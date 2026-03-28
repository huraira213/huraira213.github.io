---
layout: default
title: PostgreSQL Query Profiler
---

<div class="page-header">
  <h1>PostgreSQL Query Profiler</h1>
  <p class="tagline">Continuous Query Observability with Delta Metrics</p>
</div>

<div class="container">

<div class="terminal-section">
  <div class="terminal-header">
    <span class="terminal-title">$ ./profiler --version</span>
  </div>
  <div class="terminal-body">
    <p><strong>Project:</strong> PostgreSQL Query Profiler</p>
    <p><strong>Status:</strong> Production-Ready</p>
    <p><strong>Stack:</strong> Python 3.10+, pg_stat_statements, psycopg2</p>
    <p><strong>Source:</strong> <a href="https://github.com/huraira213/PostgreSQL-Query-Profile">github.com/huraira213/PostgreSQL-Query-Profile</a></p>
    
<div class="stats-grid" style="margin-top: 2rem;">
  <div class="stat-card">
    <span class="stat-number">∆</span>
    <span class="stat-label">Delta Metrics</span>
  </div>
  <div class="stat-card">
    <span class="stat-number">4</span>
    <span class="stat-label">Normalized Tables</span>
  </div>
  <div class="stat-card">
    <span class="stat-number">30%</span>
    <span class="stat-label">Regression Threshold</span>
  </div>
  <div class="stat-card">
    <span class="stat-number">60s</span>
    <span class="stat-label">Default Interval</span>
  </div>
</div>

  </div>
</div>

<section>
<h2>What It Does</h2>

A production-grade foundation for PostgreSQL query observability built on `pg_stat_statements`. Converts cumulative PostgreSQL statistics into actionable, time-bounded interval metrics with regression detection.

**Key Features:**
- Continuous snapshot collection from `pg_stat_statements`
- Delta metric computation between snapshots
- Regression detection based on historical baselines
- Normalized storage schema (snapshots, queries, metrics, deltas)
- Cross-version compatibility (PostgreSQL 12-16)
- Retention policy to prevent table bloat

<div class="project-tags">
  <span class="tag">Delta Modeling</span>
  <span class="tag">Restart Detection</span>
  <span class="tag">Baseline Analysis</span>
  <span class="tag">Cross-Version Compat</span>
</div>

</section>

<section>
<h2>PostgreSQL Internals Demonstrated</h2>

### 1. Cumulative Counter to Interval Delta Conversion

**The Problem:**
`pg_stat_statements` provides *cumulative* counters since last reset:
```sql
SELECT queryid, calls, total_exec_time 
FROM pg_stat_statements 
WHERE queryid = 12345;

-- Returns: 
-- queryid | calls | total_exec_time
-- 12345   | 1000  | 450000.0  (cumulative since last reset)
```

You can't answer: "How many calls in the last minute?"

**The Solution:**
```python
# Snapshot N-1 (previous)
prev_calls = 1000
prev_exec_time = 450000.0

# Snapshot N (current)
curr_calls = 1050
curr_exec_time = 458000.0

# Delta computation
delta_calls = curr_calls - prev_calls  # 50 calls
delta_exec_time = curr_exec_time - prev_exec_time  # 8000.0 ms

# Derived metrics
calls_per_second = delta_calls / interval_seconds  # 50 / 60 = 0.83
mean_latency = delta_exec_time / delta_calls  # 8000 / 50 = 160 ms
```

**Why This Matters:**
- Spot sudden workload changes ("calls jumped from 10/sec → 100/sec")
- Detect query regressions ("latency increased 50% this hour")
- Time-window analysis for dashboards

**PostgreSQL Concept:** Converting cumulative stats to interval metrics

---

### 2. PostgreSQL Restart Detection

**The Problem:**
When PostgreSQL restarts, `pg_stat_statements` counters reset to zero:
```python
# Before restart
prev_calls = 1000

# After restart
curr_calls = 50  # Reset!

# Naive delta
delta = curr_calls - prev_calls  # -950 (WRONG!)
```

Negative deltas are invalid and corrupt analysis.

**The Solution:**
```python
def detect_restart(prev_metrics, curr_metrics):
    """
    Restart detected if ANY cumulative counter decreased
    """
    restart_signals = [
        curr_metrics['calls'] < prev_metrics['calls'],
        curr_metrics['total_exec_time'] < prev_metrics['total_exec_time'],
        curr_metrics['shared_blks_read'] < prev_metrics['shared_blks_read'],
    ]
    
    return any(restart_signals)

# Delta computation with restart guard
if detect_restart(prev, curr):
    # Restart detected - can't compute valid delta
    delta_calls = None  # Mark as invalid
    delta_exec_time = None
else:
    # Normal case
    delta_calls = max(0, curr['calls'] - prev['calls'])
    delta_exec_time = max(0, curr['total_exec_time'] - prev['total_exec_time'])
```

**Why This Matters:**
- Prevents false "massive latency drop" signals after restart
- Maintains data integrity across PostgreSQL restarts
- Explicit handling better than silent corruption

**PostgreSQL Concept:** Understanding cumulative counter reset behavior

---

### 3. Cross-Version `pg_stat_statements` Compatibility

**The Problem:**
Column names changed in PostgreSQL 13:

| PostgreSQL 12 | PostgreSQL 13+ |
|---------------|----------------|
| `total_time` | `total_exec_time` |
| `mean_time` | `mean_exec_time` |

Hardcoded queries break across versions.

**The Solution:**
```python
def introspect_pg_stat_statements_columns(connection):
    """
    Detect available columns at runtime
    """
    query = """
        SELECT column_name 
        FROM information_schema.columns 
        WHERE table_schema = 'public'
          AND table_name = 'pg_stat_statements'
    """
    
    result = connection.execute(query).fetchall()
    columns = [row['column_name'] for row in result]
    
    return columns

def build_collection_query(columns):
    """
    Generate version-safe SELECT list
    """
    # Map to version-appropriate names
    time_col = 'total_exec_time' if 'total_exec_time' in columns else 'total_time'
    mean_col = 'mean_exec_time' if 'mean_exec_time' in columns else 'mean_time'
    
    query = f"""
        SELECT 
            queryid,
            query,
            calls,
            {time_col} as total_exec_time,
            {mean_col} as mean_exec_time,
            rows,
            shared_blks_hit,
            shared_blks_read
        FROM pg_stat_statements
        WHERE queryid IS NOT NULL
    """
    
    return query
```

**Why This Matters:**
- Single codebase supports PostgreSQL 12-16
- No version detection hacks
- Graceful degradation for missing columns

**PostgreSQL Concept:** System catalog introspection via `information_schema`

</section>

<section>
<h2>Architecture</h2>

<div class="architecture">
pg_stat_statements (cumulative counters)
      ↓
Collector Service
      ↓
  ┌───────────────────────────────┐
  │  Snapshot N                   │
  │  - timestamp: 10:00:00        │
  │  - interval: 60s              │
  │  ├─ Query A: calls=1050       │
  │  ├─ Query B: calls=500        │
  │  └─ Query C: calls=20         │
  └───────────────────────────────┘
      ↓
Persist to Database
      ├─> snapshots (id, timestamp, interval)
      ├─> queries (queryid, query_text)
      └─> query_metrics (snapshot_id, queryid, calls, exec_time, ...)
      ↓
Load Previous Snapshot (N-1)
      ↓
Restart Detection
      ├─> Check if counters decreased
      ├─> If yes: skip delta (invalid)
      └─> If no: proceed to delta
      ↓
Delta Computation
      ├─> delta_calls = curr_calls - prev_calls
      ├─> delta_exec_time = curr_exec_time - prev_exec_time
      └─> Clamp at zero (max(0, delta))
      ↓
Persist Delta Metrics
      └─> delta_metrics (snapshot_id, queryid, delta_calls, ...)
      ↓
Regression Detection
      ├─> Load baseline (avg of last N deltas)
      ├─> Compare current vs baseline
      ├─> If deviation > threshold: regression
      └─> Store in regression_events
      ↓
Analysis/Export/Visualization
      ├─> CLI: analyze, export, visualize
      ├─> Sort by: total_exec_time, calls, io_cost
      └─> Output: CSV, JSON, terminal tables
</div>

</section>

<section>
<h2>What Broke (and How I Fixed It)</h2>

### Problem 1: Cross-Version Column Name Differences

<div class="error-box">
<strong>What Happened:</strong>
```python
# WRONG - Hardcoded PostgreSQL 13+ column names
query = """
    SELECT 
        queryid,
        total_exec_time,  -- Doesn't exist in PG 12!
        mean_exec_time
    FROM pg_stat_statements
"""

# Error on PostgreSQL 12:
# column "total_exec_time" does not exist
```

Tool worked on PostgreSQL 13+ but failed on 12 with cryptic column errors.
</div>

**Debug Process:**
1. User reported: "Profiler crashes on PostgreSQL 12"
2. Connected to PG 12: `\d pg_stat_statements`
3. Noticed: `total_time` vs `total_exec_time`
4. Realized: Column rename in PG 13

**Fix:**
```python
# RIGHT - Dynamic column detection
def get_stat_statements_schema(connection):
    query = """
        SELECT column_name 
        FROM information_schema.columns 
        WHERE table_name = 'pg_stat_statements'
    """
    columns = connection.execute(query).fetchall()
    return [row['column_name'] for row in columns]

def build_query(columns):
    # Adapt to available columns
    if 'total_exec_time' in columns:
        time_col = 'total_exec_time'
    else:
        time_col = 'total_time'
    
    return f"SELECT queryid, {time_col} as total_exec_time, ..."
```

**Lesson:** PostgreSQL evolves. Never hardcode version-specific features.

---

### Problem 2: Negative Deltas from Counter Resets

<div class="error-box">
<strong>What Happened:</strong>
```python
# WRONG - No restart detection
delta_calls = curr_calls - prev_calls

# After PostgreSQL restart:
# prev_calls = 1000 (before restart)
# curr_calls = 50 (after restart - counters reset!)
# delta_calls = 50 - 1000 = -950 (INVALID!)
```

Regression detector flagged: "Query performance improved 95%!" (false positive)
</div>

**Debug Process:**
1. Noticed impossible delta values in logs
2. Correlated with PostgreSQL restart timestamps
3. Realized: `pg_stat_statements` resets on restart
4. Checked PostgreSQL docs: "Statistics are lost on server restart"

**Fix:**
```python
# RIGHT - Single-pass restart detection
def detect_restart(prev_metrics, curr_metrics):
    # Check ALL cumulative counters
    counters = [
        'calls',
        'total_exec_time',
        'rows',
        'shared_blks_hit',
        'shared_blks_read',
        'shared_blks_dirtied',
        'shared_blks_written'
    ]
    
    for counter in counters:
        if curr_metrics[counter] < prev_metrics[counter]:
            return True  # Restart detected
    
    return False

# Delta computation
if detect_restart(prev, curr):
    # Skip delta - invalid baseline
    delta = None
else:
    # Normal delta with safety clamp
    delta = max(0, curr - prev)
```

**Lesson:** Cumulative counters can decrease. Always detect and handle resets.

---

### Problem 3: Noisy SQL Polluting Rankings

<div class="warning-box">
<strong>What Happened:</strong>
```python
# Top queries by execution time:
# 1. BEGIN
# 2. COMMIT
# 3. SET application_name = ...
# 4. (actual slow query hidden on page 2)
```

Transaction control and DDL statements dominated the top-N, hiding actual slow queries.
</div>

**Fix:**
```python
# Noise filter before persistence
NOISE_PATTERNS = [
    r'^BEGIN',
    r'^COMMIT',
    r'^ROLLBACK',
    r'^SET\s+',
    r'^SHOW\s+',
    r'^CREATE\s+EXTENSION',
    r'^DROP\s+EXTENSION',
    r'^VACUUM',
    r'^ANALYZE',
]

def is_noise_query(query_text):
    for pattern in NOISE_PATTERNS:
        if re.match(pattern, query_text, re.IGNORECASE):
            return True
    return False

# Collection filtering
for row in pg_stat_statements_results:
    if not is_noise_query(row['query']):
        persist_metric(row)
```

**Lesson:** Raw `pg_stat_statements` contains system/maintenance queries. Filter before analysis.

</section>

<section>
<h2>Data Model</h2>

### Normalized Schema
```sql
-- Snapshot metadata
CREATE TABLE snapshots (
    id SERIAL PRIMARY KEY,
    collected_at TIMESTAMPTZ NOT NULL,
    database_name TEXT NOT NULL,
    interval_seconds INT NOT NULL
);

-- Query definitions (deduped)
CREATE TABLE queries (
    queryid BIGINT PRIMARY KEY,
    query_text TEXT NOT NULL,
    first_seen TIMESTAMPTZ NOT NULL,
    last_seen TIMESTAMPTZ NOT NULL
);

-- Raw cumulative metrics
CREATE TABLE query_metrics (
    snapshot_id INT REFERENCES snapshots(id),
    queryid BIGINT REFERENCES queries(queryid),
    calls BIGINT,
    total_exec_time DOUBLE PRECISION,
    mean_exec_time DOUBLE PRECISION,
    rows BIGINT,
    shared_blks_hit BIGINT,
    shared_blks_read BIGINT,
    PRIMARY KEY (snapshot_id, queryid)
);

-- Interval delta metrics
CREATE TABLE delta_metrics (
    snapshot_id INT REFERENCES snapshots(id),
    queryid BIGINT REFERENCES queries(queryid),
    delta_calls BIGINT,
    delta_exec_time DOUBLE PRECISION,
    delta_rows BIGINT,
    delta_shared_blks_hit BIGINT,
    delta_shared_blks_read BIGINT,
    PRIMARY KEY (snapshot_id, queryid)
);

-- Regression events
CREATE TABLE regression_events (
    snapshot_id INT REFERENCES snapshots(id),
    queryid BIGINT REFERENCES queries(queryid),
    metric TEXT,  -- 'mean_exec_time', 'calls_per_sec', etc.
    baseline_value DOUBLE PRECISION,
    current_value DOUBLE PRECISION,
    pct_change DOUBLE PRECISION,
    detected_at TIMESTAMPTZ DEFAULT NOW()
);
```

**Indexes:**
```sql
CREATE INDEX idx_snapshots_collected_at ON snapshots(collected_at DESC);
CREATE INDEX idx_query_metrics_queryid ON query_metrics(queryid);
CREATE INDEX idx_delta_metrics_queryid ON delta_metrics(queryid);
```

</section>

<section>
<h2>Production-Ready Features Missing</h2>

### 1. Alerting/Notification Delivery

**Current State:** Regressions stored in database  
**Production Need:**
- Threshold policies per workload
- Outbound integrations (Slack/Email/PagerDuty)
- Deduplication windows (don't alert same regression every minute)
- Incident audit trail

---

### 2. Query Plan Capture

**Current State:** Only timing/call stats  
**Production Need:**
- EXPLAIN plan capture alongside metrics
- Plan drift detection (query started using different plan)
- Index recommendations from plan analysis

---

### 3. Multi-Database Monitoring

**Current State:** Single database per run  
**Production Need:**
- Monitor all databases in a cluster
- Cross-database workload comparison
- Shared regression detection across databases

</section>

<section>
<h2>Key Learnings</h2>

### 1. Cumulative Counters Require Delta Modeling
`pg_stat_statements` gives lifetime totals. Interval analysis requires:
- Snapshot storage
- Delta computation
- Restart detection

### 2. PostgreSQL Internals Evolve
Column renames, new views, feature additions happen across versions.  
**Never hardcode** - always introspect.

### 3. Noise Filtering is Essential
Raw `pg_stat_statements` contains everything:
- Transaction control
- Maintenance statements
- Extension DDL

Filter noise before storage to save space and improve signal.

### 4. Regression Detection Needs Baselines
Comparing current snapshot to previous is noisy.  
Better: Compare to rolling average of last N intervals.

</section>

<section>
<h2>Usage Example</h2>
```bash
# Initialize schema
python main.py init-db

# Collect one snapshot
python main.py collect

# Run continuous collection (every 60s)
python main.py run --interval 60

# Analyze latest delta metrics
python main.py analyze --mode delta --sort total_exec_time --limit 20

# Output:
# query_text                          | calls | mean_time | total_time | rows | io_cost | score
# SELECT * FROM large_table WHERE ... | 120   | 450.2     | 54024.0    | 900  | 32000   | 0.93
# UPDATE users SET last_login = ...   | 85    | 380.5     | 32342.5    | 85   | 18000   | 0.87

# Export to CSV
python main.py export --format csv --path ./reports --mode delta

# Visualize slowest queries
python main.py visualize --mode slowest --limit 10

# Check for regressions
SELECT * FROM regression_events 
WHERE detected_at > NOW() - INTERVAL '1 hour'
ORDER BY pct_change DESC;

# Cleanup old data
python main.py purge --days 30
```

</section>

<section>
<h2>Resources Used for Learning</h2>

**`pg_stat_statements` Internals:**
- [PostgreSQL pg_stat_statements Docs](https://www.postgresql.org/docs/current/pgstatstatements.html)
- Source code: `contrib/pg_stat_statements/pg_stat_statements.c`

**Delta Metric Modeling:**
- Prometheus query patterns (rate/irate functions)
- Time-series database design patterns

**Cross-Version Compatibility:**
- PostgreSQL release notes (12, 13, 14, 15, 16)
- `information_schema` catalog documentation

**Regression Detection:**
- Statistical process control (baseline + threshold)
- Anomaly detection in time series

</section>

<section>
<h2>What I'd Do Differently Now</h2>

1. **Query Fingerprints** - Store normalized query hash in addition to raw text
2. **Transaction Boundaries** - Wrap snapshot+delta+regression in single transaction
3. **Per-Database Partitioning** - Partition metrics by database_name for scaling
4. **Workload Tags** - Allow user-defined tags for queries (backend, frontend, batch)
5. **Test Coverage** - Integration tests with ephemeral PostgreSQL from day 1

</section>

</div>

<footer>
  <p><a href="../index.html">← Back to Projects</a></p>
  <p>Huraira Khurshid | Database Engine Developer</p>
</footer>