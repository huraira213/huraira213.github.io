---
layout: default
title: Currently Learning
---

<div class="page-header">
  <h1>Currently Learning</h1>
  <p class="tagline">What I'm Studying at Inzent</p>
</div>

<div class="container">

<div class="terminal-section">
  <div class="terminal-header">
    <span class="terminal-title">$ tail -f ~/learning.log</span>
  </div>
  <div class="terminal-body">
    <p><strong>Updated:</strong> March 2026</p>
    <p><strong>Current Role:</strong> Database Engine Developer at Inzent</p>
    <p><strong>Focus:</strong> PostgreSQL diagnostic tooling → C-based core extensions</p>
  </div>
</div>

<section>
<h2>Active Learning (This Month)</h2>

### 1. Diagnostic Pipeline Reliability
**Context:** Building `lasso_collector` at Inzent

**What I'm Learning:**
- Deterministic artifact generation under failure conditions
- Subprocess timeout handling with consistent output
- CLI contract enforcement (`--dry-run`, `--skip-queries`)
- Error taxonomy for operational diagnostics

**Recent Challenge:**
Timeout paths were non-deterministic - if a collector timed out, no artifact was written. Fixed by implementing graceful degradation: write partial artifact with timeout metadata.

**Resources:**
- Python `subprocess` module documentation
- "The Practice of Programming" (error handling chapter)
- EDB LASSO design patterns (inspiration)

---

### 2. Transaction Snapshot Semantics
**Context:** Already implemented in `pg_diag`, deepening understanding

**What I'm Learning:**
- `REPEATABLE READ` vs `SERIALIZABLE` isolation levels
- Snapshot visibility rules in MVCC
- Transaction rollback patterns after errors
- When to use `READ ONLY` transactions

**Key Insight:**
PostgreSQL transactions don't auto-recover from errors. After any SQL error, the transaction enters "abort" state and all subsequent commands fail until `ROLLBACK`.

**Next Steps:**
- Study PostgreSQL source: `src/backend/access/transam/xact.c`
- Read "Transaction Processing" (Jim Gray) - Chapter 3

---

### 3. System Catalog Introspection
**Context:** Cross-version compatibility in diagnostic tools

**What I'm Learning:**
- Dynamic query generation from `information_schema`
- Column existence checks before SELECT
- Capability detection patterns
- Version-aware feature gating

**Example:**
```python
# Instead of hardcoding PostgreSQL 13+ column names
query = "SELECT total_exec_time FROM pg_stat_statements"

# Introspect at runtime
columns = get_available_columns('pg_stat_statements')
time_col = 'total_exec_time' if 'total_exec_time' in columns else 'total_time'
query = f"SELECT {time_col} as total_exec_time FROM pg_stat_statements"
```

**Resources:**
- PostgreSQL `information_schema` docs
- pg_stat_statements source code (cross-version diffs)

</section>

<section>
<h2>Deep Dive Queue (Next 3 Months)</h2>

### 1. PostgreSQL Executor Internals
**Goal:** Prepare for C-based extension work

**Plan:**
- Read `src/backend/executor/execMain.c`
- Understand executor hook lifecycle
- Study `ExecutorStart`, `ExecutorRun`, `ExecutorFinish`, `ExecutorEnd`
- Build simple executor hook for query logging

**Why This Matters:**
Next project at Inzent likely involves C-based instrumentation. Executor hooks are the foundation for query tracing and performance monitoring.

**Resources:**
- PostgreSQL 14 Internals (Part III: Query Execution)
- Hironobu Suzuki's executor diagrams
- `pg_stat_statements` source (uses executor hooks)

---

### 2. Memory Context Deep Dive
**Goal:** Master PostgreSQL memory management

**Current Understanding:**
- Basic palloc/pfree usage
- Context hierarchy (TopMemoryContext → session → transaction → query)
- Why malloc is forbidden

**Gap:**
- When to create custom contexts
- Context callbacks for cleanup
- Debugging memory context leaks

**Learning Plan:**
- Read `src/backend/utils/mmgr/mcxt.c` line by line
- Build extension with custom memory context
- Use `MemoryContextStats()` for debugging

---

### 3. Varlena Internals
**Goal:** Understand PostgreSQL variable-length types

**What I Know:**
- Basic varlena structure (header + data)
- `VARSIZE_ANY_EXHDR`, `SET_VARSIZE`, `VARDATA_ANY`

**What I Don't Know:**
- TOAST (The Oversized-Attribute Storage Technique)
- Compressed varlena
- External storage for large values
- Alignment requirements

**Plan:**
- Read `src/backend/utils/adt/varlena.c`
- Study TOAST code: `src/backend/access/heap/tuptoaster.c`
- Build extension that handles TOAST values

</section>

<section>
<h2>Learning Goals (6 Month Horizon)</h2>

<div class="timeline">
  <div class="timeline-item">
    <h3>Q2 2026: Master Executor Hooks</h3>
    <p><strong>Deliverable:</strong> Build production-ready query tracing extension</p>
    <p><strong>Skills:</strong> Hook registration, memory context isolation, recursive tracing prevention</p>
  </div>
  
  <div class="timeline-item">
    <h3>Q2 2026: MVCC Understanding</h3>
    <p><strong>Deliverable:</strong> Write technical blog post explaining MVCC</p>
    <p><strong>Skills:</strong> Transaction isolation, snapshot visibility, tuple versioning</p>
  </div>
  
  <div class="timeline-item">
    <h3>Q3 2026: Query Planner Basics</h3>
    <p><strong>Deliverable:</strong> Explain EXPLAIN output at systems level</p>
    <p><strong>Skills:</strong> Cost estimation, join algorithms, index selection</p>
  </div>
  
  <div class="timeline-item">
    <h3>Q3 2026: First pgsql-hackers Contribution</h3>
    <p><strong>Deliverable:</strong> Submit patch (even if rejected)</p>
    <p><strong>Skills:</strong> PostgreSQL development workflow, code review, community interaction</p>
  </div>
</div>

</section>

<section>
<h2>Current Reading List</h2>

**In Progress:**
1. **PostgreSQL 14 Internals** (Egor Rogov) - Part III: Query Execution
2. **Expert C Programming** (Peter van der Linden) - Chapters 5-8
3. **pgsql-hackers Archives** - Following patch discussions on query optimizer

**Queued:**
1. **Transaction Processing** (Jim Gray) - MVCC and isolation levels
2. **Database Internals** (Alex Petrov) - Storage engine design
3. **The Art of PostgreSQL** (Dimitri Fontaine) - Query optimization patterns

</section>

<section>
<h2>Practice Projects (Side Learning)</h2>

### 1. pg_explain_annotated
**Goal:** Understand query planner output

**Plan:**
- Parse EXPLAIN JSON output
- Annotate with human-readable explanations
- Highlight potential issues (seq scans on large tables, expensive sorts)

**Learning:**
- Query planner internals
- Cost estimation formulas
- Index selection logic

---

### 2. Simple MVCC Implementation
**Goal:** Build toy database with MVCC

**Plan:**
- Implement tuple versioning
- Write basic transaction manager
- Support READ COMMITTED and REPEATABLE READ

**Learning:**
- Transaction ID management
- Visibility rules
- Vacuum-like cleanup

**Status:** Design phase

---

### 3. WAL Parser
**Goal:** Understand write-ahead logging

**Plan:**
- Parse PostgreSQL WAL files
- Display operations (INSERT, UPDATE, DELETE)
- Understand checkpointing

**Learning:**
- Crash recovery
- Replication internals
- REDO/UNDO logging

**Status:** Research phase

</section>

<section>
<h2>Recent Insights</h2>

### Insight 1: Error Handling is Architecture
**Learned:** How you handle errors determines system reliability.

**Example:**
In `pg_diag`, I learned that partial success (collecting 45/50 metrics) is often more valuable than all-or-nothing. Proper error taxonomy (success/partial/skipped/error) helps operators debug issues.

**Applied to:** `lasso_collector` design - timeouts write partial artifacts with metadata.

---

### Insight 2: Cross-Version Compatibility Requires Introspection
**Learned:** Never hardcode version-specific features.

**Example:**
PostgreSQL 13 renamed `total_time` → `total_exec_time` in `pg_stat_statements`. Hardcoded column names break on older versions.

**Solution:** Query `information_schema.columns` at runtime and adapt.

---

### Insight 3: Memory Contexts Are Powerful
**Learned:** PostgreSQL's memory context system prevents entire classes of bugs.

**Compared to C:**
- C: Every malloc needs matching free (easy to forget)
- PostgreSQL: Allocate in context, destroy context later (automatic cleanup)

**Gotcha:** Must understand context lifetimes or you'll leak anyway.

</section>

<section>
<h2>Learning Metrics (Self-Assessment)</h2>

<div class="skills-grid">
  <div class="skill-category">
    <h4>PostgreSQL Internals</h4>
    <ul>
      <li>Extension Development: Advanced</li>
      <li>Memory Contexts: Intermediate</li>
      <li>Executor Hooks: Intermediate</li>
      <li>SPI: Advanced</li>
      <li>Query Planner: Beginner</li>
      <li>MVCC: Intermediate</li>
    </ul>
  </div>
  
  <div class="skill-category">
    <h4>C Programming</h4>
    <ul>
      <li>Pointers: Advanced</li>
      <li>Memory Mgmt: Advanced</li>
      <li>Debugging (GDB): Intermediate</li>
      <li>Valgrind: Intermediate</li>
      <li>System Calls: Beginner</li>
    </ul>
  </div>
  
  <div class="skill-category">
    <h4>Systems Concepts</h4>
    <ul>
      <li>Process Models: Intermediate</li>
      <li>IPC: Beginner</li>
      <li>File I/O: Intermediate</li>
      <li>Concurrency: Beginner</li>
    </ul>
  </div>
</div>

**Legend:** Beginner | Intermediate | Advanced | Expert

</section>

<section>
<h2>Weekly Learning Routine</h2>

**Monday-Friday (During Work):**
- Morning: Work on current Inzent project (`lasso_collector`)
- Lunch: Read PostgreSQL source code (30 min)
- Evening: Technical reading (1 hour)

**Saturday:**
- Deep dive session (4 hours)
- Focus: One complex PostgreSQL internals topic
- Output: Notes + code examples

**Sunday:**
- Practice project (2-3 hours)
- Build small extension or tool
- Document learnings

**Monthly:**
- Review progress against goals
- Update this page
- Adjust learning plan based on Inzent project needs

</section>

<section>
<h2>Questions I'm Trying to Answer</h2>

1. **How does PostgreSQL decide whether to use an index?**
   - Status: Reading query planner code
   - Blocker: Need to understand cost estimation formulas

2. **What happens during VACUUM at a low level?**
   - Status: Haven't started
   - Plan: Read vacuum code after mastering MVCC

3. **How does replication work under the hood?**
   - Status: High-level understanding only
   - Plan: Build WAL parser to understand format

4. **When should I create a custom memory context?**
   - Status: Basic understanding
   - Blocker: Need more real-world examples

5. **How does TOAST handle large values?**
   - Status: Conceptual understanding
   - Plan: Read tuptoaster.c and build test extension

</section>

</div>

<footer>
  <p><a href="../index.html">← Back to Home</a></p>
  <p>Huraira Khurshid | Database Engine Developer</p>
</footer>
