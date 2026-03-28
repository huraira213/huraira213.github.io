---
layout: default
title: pg_diag - PostgreSQL Diagnostic Collector
---

<div class="page-header">
  <h1>pg_diag</h1>
  <p class="tagline">Production-Grade PostgreSQL Diagnostic Collector</p>
</div>

<div class="container">

<div class="terminal-section">
  <div class="terminal-header">
    <span class="terminal-title">$ ./pg_diag --version</span>
  </div>
  <div class="terminal-body">
    <p><strong>Project:</strong> pg_diag</p>
    <p><strong>Status:</strong> Production (Built at Inzent)</p>
    <p><strong>Stack:</strong> Python 3.10+, psycopg3, psutil</p>
    <p><strong>Source:</strong> <a href="https://github.com/huraira213/PG_Diag">github.com/huraira213/PG_Diag</a></p>
    
<div class="stats-grid" style="margin-top: 2rem;">
  <div class="stat-card">
    <span class="stat-number">50+</span>
    <span class="stat-label">Collectors</span>
  </div>
  <div class="stat-card">
    <span class="stat-number">3</span>
    <span class="stat-label">Depth Levels</span>
  </div>
  <div class="stat-card">
    <span class="stat-number">100%</span>
    <span class="stat-label">Snapshot Consistent</span>
  </div>
  <div class="stat-card">
    <span class="stat-number">4</span>
    <span class="stat-label">Status Types</span>
  </div>
</div>

  </div>
</div>

<section>
<h2>What It Does</h2>

pg_diag is a comprehensive PostgreSQL diagnostic collection tool that gathers system and database state information for troubleshooting, performance tuning, and health monitoring. Built at Inzent for production operational diagnostics.

**Key Features:**
- Snapshot-consistent collection using `REPEATABLE READ` transaction semantics
- 50+ collectors across OS, PostgreSQL, HA tools, and backup systems
- Depth control: `surface`, `shallow`, `deep` execution levels
- Graceful degradation with partial/error status taxonomy
- Automatic secret redaction for safe sharing
- Modular collector architecture

<div class="project-tags">
  <span class="tag">REPEATABLE READ</span>
  <span class="tag">Capability Detection</span>
  <span class="tag">Graceful Degradation</span>
  <span class="tag">Error Taxonomy</span>
  <span class="tag">Secret Redaction</span>
</div>

</section>

<section>
<h2>PostgreSQL Internals Demonstrated</h2>

### 1. Snapshot-Consistent Transaction Semantics

**The Problem:**
When collecting 50+ metrics, you need them from the *same point in time*. Without snapshot isolation, metrics can be inconsistent (e.g., connection count changes between queries).

**The Solution:**
```python
# Engine opens REPEATABLE READ transaction
connection.execute("BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ READ ONLY")

try:
    # All DB collectors run inside this snapshot
    for collector in db_collectors:
        result = collector.collect(context)
        
    connection.execute("COMMIT")
except Exception as e:
    connection.execute("ROLLBACK")
```

**Why This Matters:**
- All `pg_stat_*` views are queried at the same snapshot
- Lock state, replication lag, connection counts are consistent
- No mid-collection changes corrupt the diagnostic picture

**PostgreSQL Concept:** Transaction snapshot isolation via MVCC

---

### 2. Capability Detection for Version-Aware Collection

**The Problem:**
`pg_stat_statements` exists in PostgreSQL 12+ but columns differ by version. `pg_replication_slots` has different fields in PG 13 vs 16.

**The Solution:**
```python
class CapabilityDetector:
    def detect_pg_stat_statements(self):
        # Check extension exists
        query = "SELECT 1 FROM pg_extension WHERE extname = 'pg_stat_statements'"
        if not self._query_exists(query):
            return False
        
        # Introspect available columns
        query = """
            SELECT column_name 
            FROM information_schema.columns 
            WHERE table_name = 'pg_stat_statements'
        """
        columns = self._fetch_columns(query)
        
        # Adapt query to available columns
        if 'total_exec_time' in columns:
            # PostgreSQL 13+
            return 'total_exec_time'
        elif 'total_time' in columns:
            # PostgreSQL 12
            return 'total_time'
```

**Why This Matters:**
- Same tool works across PostgreSQL 12-16
- No hardcoded version checks
- Graceful feature degradation when extension missing

**PostgreSQL Concept:** System catalog introspection via `information_schema`

---

### 3. Error Taxonomy with Partial Success

**The Problem:**
Production systems often have permission issues, missing extensions, inaccessible files. A single failure shouldn't abort the entire diagnostic run.

**The Solution:**
```python
class BaseCollector:
    def execute(self, context):
        start_time = time.time()
        
        try:
            result = self.collect(context)
            
            # Check for embedded errors in result payload
            if self._has_embedded_errors(result):
                status = "partial"  # Data collected, but with issues
            else:
                status = "success"
                
        except PermissionError as e:
            result = {"error": str(e)}
            status = "error"
        except ExtensionNotFound as e:
            result = {"skipped": str(e)}
            status = "skipped"
        
        return {
            "status": status,
            "data": result,
            "duration": time.time() - start_time
        }
```

**Status Model:**
- `success` - Data collected without errors
- `partial` - Data collected with some embedded errors
- `skipped` - Collector intentionally skipped (missing extension, wrong depth)
- `error` - Complete failure

**Why This Matters:**
- One broken collector doesn't break the entire run
- Summary report shows: 45 success, 3 partial, 2 skipped, 0 error
- Diagnostic still useful even with partial data

**PostgreSQL Concept:** Defensive programming for operational tooling

</section>

<section>
<h2>Architecture</h2>

<div class="architecture">
CLI Arguments
    ↓
Config Loader (env + CLI overrides)
    ↓
Context Initialization
    ↓
Capability Detector
    ├─> PostgreSQL version
    ├─> Extension availability (pg_stat_statements)
    ├─> Replication mode (primary/standby)
    └─> HA tool detection (Patroni/repmgr/PgBouncer)
    ↓
Engine Execution
    ├─> OS Collectors (no DB required)
    │   ├─> system_info
    │   ├─> disk_usage
    │   ├─> postgres_processes
    │   └─> kernel_tuning
    │
    └─> DB Collectors (inside REPEATABLE READ snapshot)
        ├─> pg_version
        ├─> pg_settings
        ├─> pg_connections
        ├─> pg_locks
        ├─> pg_replication
        ├─> pg_cache_hit
        ├─> pg_bloat
        └─> pg_stat_statements
    ↓
File Writer
    ├─> diagnostic_report.json (all results)
    ├─> summary.json (status counts)
    ├─> metadata.json (run context)
    └─> individual collector files (postgres/*, system/*)
</div>

</section>

<section>
<h2>What Broke (and How I Fixed It)</h2>

### Problem 1: Transaction Poisoning from Collector Errors

<div class="error-box">
<strong>What Happened:</strong>
```python
# WRONG - One error aborts entire transaction
connection.execute("BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ")

for collector in db_collectors:
    result = collector.collect(connection)  # Error here!
    # Transaction now in ABORT state
    # All subsequent collectors fail
```

First collector raises `ProgrammingError` → transaction enters abort state → all remaining collectors fail with "current transaction is aborted".
</div>

**Debug Process:**
1. Noticed cascading failures in logs
2. All collectors after first error showed "transaction aborted"
3. Realized PostgreSQL transaction state machine doesn't auto-recover

**Fix:**
```python
# RIGHT - Explicit rollback and fallback
connection.execute("BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ")

try:
    for collector in db_collectors:
        result = collector.collect(connection)
    connection.execute("COMMIT")
    
except Exception as e:
    connection.execute("ROLLBACK")
    logger.warning("Snapshot mode failed, falling back to individual queries")
    
    # Run remaining collectors outside snapshot
    for collector in remaining:
        try:
            result = collector.collect_without_snapshot(connection)
        except Exception as collector_error:
            # Individual failure doesn't affect others
            logger.error(f"Collector {collector.name} failed: {collector_error}")
```

**Lesson:** PostgreSQL transactions don't auto-recover from errors. Always handle rollback explicitly.

---

### Problem 2: `pg_stat_statements` Column Compatibility

<div class="error-box">
<strong>What Happened:</strong>
```python
# WRONG - Hardcoded column names
query = """
    SELECT queryid, query, total_exec_time, calls
    FROM pg_stat_statements
    ORDER BY total_exec_time DESC
"""
# Fails on PostgreSQL 12 (uses 'total_time' not 'total_exec_time')
```

PostgreSQL 13 renamed `total_time` → `total_exec_time` and `mean_time` → `mean_exec_time`. Tool broke on older versions.
</div>

**Debug Process:**
1. User reported: "pg_stat_statements collector fails on PostgreSQL 12"
2. Checked column names: `\d pg_stat_statements`
3. Discovered naming change in PG 13

**Fix:**
```python
# RIGHT - Dynamic column detection
def build_pg_stat_statements_query(self, context):
    # Introspect available columns
    columns = context.capability.pg_stat_statements_columns
    
    # Map to version-appropriate names
    time_col = 'total_exec_time' if 'total_exec_time' in columns else 'total_time'
    mean_col = 'mean_exec_time' if 'mean_exec_time' in columns else 'mean_time'
    
    query = f"""
        SELECT 
            queryid,
            query,
            {time_col} as total_exec_time,
            {mean_col} as mean_exec_time,
            calls
        FROM pg_stat_statements
        ORDER BY {time_col} DESC
        LIMIT %(limit)s
    """
    return query
```

**Lesson:** Never hardcode PostgreSQL version-specific features. Use capability detection.

---

### Problem 3: Over-Reporting Success with Embedded Errors

<div class="warning-box">
<strong>What Happened:</strong>
```python
# WRONG - Doesn't detect embedded errors
def collect(self):
    results = {}
    
    try:
        results['disk_usage'] = get_disk_usage()
    except PermissionError:
        results['disk_usage'] = {"error": "Permission denied"}
    
    return results  # Marked as "success" even though it has errors!
```

Summary showed: "50 success, 0 errors" but 10 collectors had embedded error fields.
</div>

**Fix:**
```python
# RIGHT - Embedded error detection
class BaseCollector:
    def _has_embedded_errors(self, result):
        """Recursively check for error fields in nested dicts"""
        if isinstance(result, dict):
            if 'error' in result or 'errors' in result:
                return True
            return any(self._has_embedded_errors(v) for v in result.values())
        elif isinstance(result, list):
            return any(self._has_embedded_errors(item) for item in result)
        return False
    
    def execute(self, context):
        result = self.collect(context)
        
        # Check for embedded errors
        if self._has_embedded_errors(result):
            status = "partial"  # Data + errors
        else:
            status = "success"  # Clean data
        
        return {"status": status, "data": result}
```

**Lesson:** Success status must reflect actual data quality, not just "didn't throw exception".

</section>

<section>
<h2>Production-Ready Features Missing</h2>

### 1. Secure Remote Bundle Pipeline

**Current State:** Diagnostics written to local filesystem  
**Production Need:**
- Signed/compressed diagnostic bundles
- Optional encryption at rest and in transit
- Upload to S3/object store with retry + backoff
- Resumable uploads for large diagnostic sets
- Audit trail of who downloaded what

**Why This Matters:** In production, you can't always SSH to servers. Need secure remote collection.

---

### 2. Collector Manifest/Plugin Model

**Current State:** Collectors hardcoded in `main.py`  
**Production Need:**
```python
# Desired: Plugin discovery
collectors/
    postgres/
        pg_locks.py           # Auto-discovered
        pg_bloat.py
    custom/
        my_custom_check.py    # User-provided
```

**Why This Matters:** Users want to add custom collectors without modifying core code.

---

### 3. Differential Diagnostics

**Current State:** Full snapshot every run  
**Production Need:**
- Compare two diagnostic runs: `pg_diag diff report1.json report2.json`
- Show what changed (new locks, config changes, replication lag delta)
- Highlight regressions (cache hit ratio dropped, table bloat increased)

**Why This Matters:** "What changed between yesterday and today?" is a common debugging question.

</section>

<section>
<h2>Key Learnings</h2>

### 1. Snapshot Semantics Are Critical for Consistency
Without `REPEATABLE READ`, metrics drift during collection:
- Connection count changes mid-run
- Lock state inconsistent with query state
- Replication lag reported at different times

### 2. Graceful Degradation > Perfect Collection
Production systems are messy:
- Missing extensions
- Permission denied on files
- Partial data beats no data

### 3. Version Compatibility Requires Introspection
PostgreSQL evolves:
- Column renames (`total_time` → `total_exec_time`)
- New system views (PG 13+ features)
- Extension API changes

Never assume - always detect.

### 4. Error Taxonomy Clarifies Root Causes
"Failed" is useless. "Transaction aborted due to pg_locks timeout" is actionable.

Status model helps:
- `success` - Ship it
- `partial` - Review errors, might be OK
- `skipped` - Enable missing extension
- `error` - Fix immediately

</section>

<section>
<h2>Usage Example</h2>
```bash
# Basic diagnostic run
./pg_diag.py \
  --host localhost \
  --port 5432 \
  --database postgres \
  --user postgres \
  --depth shallow \
  --output ./diagnostics \
  --timestamp

# Output:
# ./diagnostics/2026-01-15T10-30-00Z/
#   ├── diagnostic_report.json
#   ├── summary.json
#   ├── metadata.json
#   ├── postgres/
#   │   ├── pg_settings.json
#   │   ├── pg_locks.json
#   │   ├── pg_cache_hit.json
#   │   └── ...
#   └── system/
#       ├── system_info.json
#       ├── disk_usage.json
#       └── ...

# Summary shows:
# Success: 45
# Partial: 3  (permission denied on some logs)
# Skipped: 2  (pg_stat_statements not installed)
# Error: 0
```

**Analysis workflow:**
```bash
# 1. Collect diagnostic
./pg_diag.py --depth shallow --output ./diag1

# 2. Review summary
cat diag1/summary.json | jq '.status_counts'

# 3. Check partial/error collectors
cat diag1/summary.json | jq '.collectors[] | select(.status != "success")'

# 4. Analyze specific metric
cat diag1/postgres/pg_cache_hit.json | jq '.database_cache_hit_ratio'

# 5. Share with team (after redaction verification)
tar -czf diag1.tar.gz diag1/
```

</section>

<section>
<h2>Resources Used for Learning</h2>

**PostgreSQL Transaction Isolation:**
- [PostgreSQL Transaction Isolation Docs](https://www.postgresql.org/docs/current/transaction-iso.html)
- [MVCC and Snapshots Explained](https://www.postgresql.org/docs/current/mvcc.html)

**System Catalogs:**
- [`pg_stat_*` views documentation](https://www.postgresql.org/docs/current/monitoring-stats.html)
- [`information_schema` catalog](https://www.postgresql.org/docs/current/information-schema.html)

**Defensive Programming:**
- "The Practice of Programming" by Kernighan & Pike
- PostgreSQL source: `src/backend/utils/error/elog.c`

**Diagnostics Design:**
- EDB LASSO documentation (inspiration)
- pgBadger source code (diagnostic patterns)

</section>

<section>
<h2>What I'd Do Differently Now</h2>

1. **Typed Result Schemas** - Use Pydantic/dataclasses for collector return types
2. **Separate Collection from Analysis** - Make heavy analysis optional/deferred
3. **Test Coverage from Day 1** - Integration tests with ephemeral PostgreSQL
4. **Collector Dependencies** - Explicit dependency graph (pg_locks needs pg_connections)
5. **Streaming Output** - Write results as collected, not buffered in memory

</section>

</div>

<footer>
  <p><a href="../index.html">← Back to Projects</a></p>
  <p>Huraira Khurshid | Database Engine Developer</p>
</footer>