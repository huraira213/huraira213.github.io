---
layout: default
title: pg_trace - PostgreSQL Query Tracing Extension
---

# pg_trace
## PostgreSQL Query Tracing Extension

**Source:** [GitHub - pg_trace](https://github.com/huraira213/pg_trace---PostgreSQL-Query-Tracing-Extension)  
**Stack:** C, PostgreSQL Executor Hooks, SPI, Memory Contexts  
**Status:** Production-ready (v6.0)

---

## What It Does

`pg_trace` is a C extension that hooks PostgreSQL's query executor to capture timing, normalized query text, and metadata for every executed query. It maintains an in-memory trace queue per session and provides SQL functions for analysis.

**Key Features:**
- Executor-level instrumentation via `ExecutorStart_hook` and `ExecutorEnd_hook`
- Normalized query text generation (literal and numeric masking)
- Query hash computation for fingerprinting
- Slow-query threshold detection
- Set-returning functions for trace analysis
- GUC-based configuration

---

## PostgreSQL Internals Demonstrated

### 1. Executor Hooks
```c
static ExecutorStart_hook_type prev_ExecutorStart = NULL;
static ExecutorEnd_hook_type prev_ExecutorEnd = NULL;

void _PG_init(void) {
    prev_ExecutorStart = ExecutorStart_hook;
    ExecutorStart_hook = pg_trace_executor_start;
    
    prev_ExecutorEnd = ExecutorEnd_hook;
    ExecutorEnd_hook = pg_trace_executor_end;
}
```

**Why this matters:** Executor hooks allow interception of every query at start/end without modifying PostgreSQL core.

### 2. Memory Context Management
```c
static MemoryContext pg_trace_context = NULL;

void _PG_init(void) {
    pg_trace_context = AllocSetContextCreate(
        TopMemoryContext,
        "pg_trace context",
        ALLOCSET_DEFAULT_SIZES
    );
}
```

**Why this matters:** Session-scoped context prevents memory leaks across query boundaries.

### 3. Recursive Self-Tracing Prevention
```c
static void pg_trace_executor_start(QueryDesc *queryDesc, int eflags) {
    const char *query = queryDesc->sourceText;
    
    // Skip self-tracing
    if (strstr(query, "pg_trace_log") != NULL ||
        strstr(query, "pg_trace_alerts") != NULL) {
        if (prev_ExecutorStart)
            prev_ExecutorStart(queryDesc, eflags);
        else
            standard_ExecutorStart(queryDesc, eflags);
        return;
    }
    // ... rest of tracing logic
}
```

**Why this matters:** Without this guard, extension catalog queries would recursively trigger more traces.

---

## Architecture
```
Query Execution
      │
      ▼
ExecutorStart_hook
      │
      ├─> Allocate QueryTrace in extension context
      ├─> Store query text + start timestamp
      └─> Call standard_ExecutorStart
      
Query Execution Continues...
      
      ▼
ExecutorEnd_hook
      │
      ├─> Compute duration
      ├─> Normalize query text
      ├─> Compute query hash
      ├─> Append to in-memory queue
      └─> Call standard_ExecutorEnd
      
      ▼
User calls pg_trace_get_queries()
      │
      └─> Returns queue as set-returning function
```

---

## What Broke (and How I Fixed It)

### Problem 1: SPI Writes from Executor Hooks

**What happened:**
```c
// UNSAFE - This caused deadlocks
static void pg_trace_executor_end(QueryDesc *queryDesc) {
    SPI_connect();
    SPI_execute("INSERT INTO pg_trace_log ...", false, 0);
    SPI_finish();
}
```

**Why it broke:** SPI operations inside executor hooks can cause lock conflicts and transaction state corruption.

**Fix:** Switched to in-memory queue collection with manual persistence:
```c
// Safe approach
static void pg_trace_executor_end(QueryDesc *queryDesc) {
    // Just append to queue
    append_to_trace_queue(trace);
}

// User manually persists when ready
SELECT * FROM pg_trace_get_queries();
INSERT INTO pg_trace_log (...) SELECT ...;
```

### Problem 2: Memory Lifecycle Pitfalls

**What happened:** Traced query strings were freed before analysis.

**Fix:** Explicit `pstrdup()` in extension context:
```c
trace->query_text = pstrdup(queryDesc->sourceText);
```

### Problem 3: Clock Anomalies

**What happened:** Backward system time caused negative durations.

**Fix:** Duration floor at zero with warning:
```c
duration_us = (end_time - start_time) * 1000000.0;
if (duration_us < 0) {
    elog(WARNING, "Negative duration detected");
    duration_us = 0;
}
```

---

## Production-Ready Features Missing

**Automatic Bounded Persistence:**
- Background worker for safe periodic flush
- Bounded queue with drop/evict policy under memory pressure
- Retry logic for durable writes

**Trace Sampling:**
- Configurable sample rate (e.g., 1% of queries)
- Adaptive sampling based on workload

**Plan Capture:**
- EXPLAIN output alongside timing
- Plan regression detection

---

## Key Learnings

1. **SPI is not safe in arbitrary contexts** - Executor hooks have strict constraints
2. **Memory contexts are hierarchical** - TopMemoryContext survives session, not query
3. **Self-tracing creates infinite loops** - Always guard against recursion
4. **System time is not monotonic** - Clock corrections happen; handle gracefully

---

## Usage Example
```sql
-- Enable tracing
SET pg_trace.enabled = on;
SET pg_trace.slow_threshold_ms = 50;

-- Run queries
SELECT count(*) FROM pg_class;

-- View traces
SELECT * FROM pg_trace_get_queries();

-- Manual persistence
INSERT INTO pg_trace_log (query_text, duration_us, ...)
SELECT query_text, duration_us, ... FROM pg_trace_get_queries();

-- Clear in-memory queue
SELECT pg_trace_clear();
```

---

## Resources Used

- [PostgreSQL Hooks Documentation](https://www.postgresql.org/docs/current/hooks.html)
- [Writing PostgreSQL Extensions](https://www.postgresql.org/docs/current/extend.html)
- [Memory Context Internals](https://www.postgresql.org/docs/current/xfunc-c.html#XFUNC-C-MEMORY)
- Source code: `src/backend/executor/execMain.c`

---

[← Back to Projects](../index.html#featured-projects)