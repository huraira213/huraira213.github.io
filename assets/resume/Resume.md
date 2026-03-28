---
layout: default
title: Resume - Huraira Khurshid
---

<div class="page-header">
  <h1>Huraira Khurshid</h1>
  <p class="tagline">Database Engine Developer</p>
</div>

<div class="container">

<div class="terminal-section">
  <div class="terminal-header">
    <span class="terminal-title">$ cat ~/contact.yaml</span>
  </div>
  <div class="terminal-body">
```yaml
location: Rawalpindi, Pakistan
email: hurairakhurshid4@gmail.com
github: github.com/huraira213
linkedin: linkedin.com/in/huraira-kiyani-05497429b
portfolio: huraira213.github.io
```

  </div>
</div>

<section>
<h2>Professional Summary</h2>

Database engine developer with hands-on PostgreSQL internals experience including C extension development, executor instrumentation, and diagnostic tooling architecture. Currently building PostgreSQL operational diagnostics at Inzent with focus on reliability, observability, and systems-level failure analysis. Strong foundation in memory context management, SPI integration, subprocess orchestration, and production-grade artifact generation.

</section>

<section>
<h2>Technical Experience</h2>

<div class="timeline">
  <div class="timeline-item">
    <h3>Database Engine Developer</h3>
    <p class="timeline-date">Inzent | Feb 2026 – Present</p>
    
**PostgreSQL Diagnostics Infrastructure:**
- Architected **pg_diag**: production-grade diagnostic collector with snapshot-consistent PostgreSQL internals capture across 50+ metrics (version info, settings, locks, replication, performance analysis)
- Implemented REPEATABLE READ transaction semantics for multi-query consistency with graceful degradation and partial-success taxonomy
- Built capability detection system for version-aware feature gating and extension availability checks
- Currently developing **lasso_collector**: EDB LASSO-inspired diagnostic pipeline with deterministic artifact generation, timeout handling, and CLI robustness for OS + database state collection

**Systems-Level Reliability Engineering:**
- Solved subprocess execution failure semantics with consistent artifact writing under timeout/error conditions
- Implemented secret redaction patterns for safe diagnostic sharing across untrusted boundaries
- Designed modular collector architecture enabling rapid extension addition without core pipeline changes
- Built CLI contract enforcement with `--dry-run`, `--skip-queries`, and flexible connection modes (Unix socket, TCP)
  </div>
  
  <div class="timeline-item">
    <h3>Software Engineering Intern – Database Systems</h3>
    <p class="timeline-date">SKAI Worldwide | May 2024 – Oct 2024</p>
    
**PostgreSQL Extension Development:**
- Developed **AgensAI extension** implementing AI model inference within PostgreSQL using SPI and custom data type handling
- Migrated Apache AGE NetworkX driver to AgensGraph by adapting query translation logic while preserving graph operation semantics
- Reverse-engineered pgAdmin4 architecture to study extensibility patterns in large-scale database management systems
- Containerized PostgreSQL, AgensGraph, and Apache AGE development environments reducing setup time from hours to minutes
  </div>
</div>

</section>

<section>
<h2>Technical Projects</h2>

### [pg_trace - PostgreSQL Query Tracing Extension](projects/pg-trace.html)
**C, Executor Hooks, SPI, Memory Contexts**

Production-grade C extension implementing executor-level query instrumentation with safe memory management.

**Demonstrates:** Executor hooks, memory context lifecycle, recursive self-tracing prevention

- Implemented `ExecutorStart_hook` and `ExecutorEnd_hook` for query lifecycle capture with normalized query text generation
- Designed session-scoped memory contexts (`AllocSetContextCreate`) preventing backend memory leaks across query boundaries
- Solved recursive self-tracing through explicit query-text guards for extension catalog access
- Built slow-query threshold detection with configurable GUC parameters and set-returning function interfaces

**Key Learning:** SPI writes from executor hook paths are unsafe; switched to in-memory queue with manual persistence

---

### [pg_cext - PostgreSQL Extension Development Foundation](projects/pg-cext.html)
**C, SPI, Varlena, Array API**

Comprehensive extension demonstrating PostgreSQL C API mastery including varlena handling and array operations.

**Demonstrates:** Varlena internals, array operations, SPI lifecycle, error handling

- Implemented string manipulation with varlena header optimization (1-byte vs 4-byte header for short strings)
- Built array operations using `ARR_DATA_PTR` direct memory access patterns for O(n) performance
- Integrated SPI for dynamic query execution with proper `SPI_connect()` / `SPI_finish()` lifecycle management
- Demonstrated `palloc()` memory context awareness across function boundaries preventing memory corruption

**Key Learning:** Memory allocated in PostgreSQL contexts persists beyond function scope; explicit cleanup required

---

### [pg_diag - PostgreSQL Diagnostic Collector](projects/pg-diag.html)
**Python, psycopg3, psutil**

Production-ready diagnostic tool with snapshot-consistent collection across 50+ PostgreSQL metrics. Built at Inzent.

**Demonstrates:** Transaction snapshot semantics, capability detection, graceful degradation, error taxonomy

- Architected `REPEATABLE READ, READ ONLY` snapshot semantics for multi-query consistency across system catalog views
- Implemented partial/error status taxonomy with embedded error detection for graceful degradation
- Built version-aware capability detection for extensions (`pg_stat_statements`), replication mode, and PostgreSQL feature flags
- Solved transaction rollback safety: explicit rollback logic prevents transaction poisoning after collector errors

**Key Learning:** Cross-version `pg_stat_statements` columns differ; schema introspection + dynamic query generation required

---

### [PostgreSQL Query Profiler](projects/pg-query-profiler.html)
**Python, pg_stat_statements**

Continuous query observability with delta metrics and regression detection.

**Demonstrates:** Cumulative counter conversion, restart detection, delta modeling, baseline analysis

- Converted `pg_stat_statements` cumulative counters into interval-based delta metrics for time-window analysis
- Implemented restart detection preventing negative delta artifacts after PostgreSQL service restart
- Built cross-version compatibility via `information_schema.columns` introspection and dynamic SELECT generation
- Designed normalized schema separating snapshots, query definitions, raw metrics, and delta metrics

**Key Learning:** Counter resets create negative deltas; single-pass restart detection with max() clamping required

</section>

<section>
<h2>Technical Skills</h2>

<div class="skills-grid">
  <div class="skill-category">
    <h4>Languages</h4>
    <ul>
      <li>C (PostgreSQL extensions)</li>
      <li>Python (diagnostic tooling)</li>
      <li>SQL (PostgreSQL dialect)</li>
      <li>Cypher (graph queries)</li>
    </ul>
  </div>
  
  <div class="skill-category">
    <h4>PostgreSQL Internals</h4>
    <ul>
      <li>C Extension Development</li>
      <li>Server Programming Interface (SPI)</li>
      <li>Memory Contexts</li>
      <li>Executor Hooks</li>
      <li>Varlena Handling</li>
      <li>System Catalogs</li>
      <li>pg_stat_* Views</li>
      <li>Query Optimization Basics</li>
    </ul>
  </div>
  
  <div class="skill-category">
    <h4>Systems Programming</h4>
    <ul>
      <li>Memory Management</li>
      <li>Process Models</li>
      <li>Subprocess Orchestration</li>
      <li>Error Handling</li>
      <li>Debugging (GDB, Valgrind)</li>
    </ul>
  </div>
  
  <div class="skill-category">
    <h4>Tools & Infrastructure</h4>
    <ul>
      <li>Docker (containerization)</li>
      <li>Git (version control)</li>
      <li>psycopg2/3 (DB drivers)</li>
      <li>psutil (system metrics)</li>
      <li>PGXS Build System</li>
    </ul>
  </div>
</div>

</section>

<section>
<h2>Education</h2>

**Bachelor of Science in Software Engineering**  
Virtual University of Pakistan | 2022 – Present

**Relevant Coursework:**
- Database Systems
- Operating Systems
- Data Structures & Algorithms
- Object-Oriented Programming
- Software Architecture

</section>

<section>
<h2>Community & Learning</h2>

- Active in PostgreSQL internals communities (r/PostgreSQL, pgsql-hackers mailing list)
- Studying PostgreSQL source code: executor (`src/backend/executor/`), memory contexts, SPI internals
- Contributing to open-source PostgreSQL ecosystem through diagnostic tooling and extension development
- Learning path focused on query planner internals, MVCC implementation, and WAL architecture

</section>

<section>
<h2>Download Resume</h2>

<div class="info-box">
<p><strong>PDF Version:</strong> <a href="assets/resume/Huraira_Khurshid_Resume.pdf">Download Resume (PDF)</a></p>
<p><strong>Last Updated:</strong> March 2026</p>
</div>

</section>

</div>

<footer>
  <p><a href="index.html">← Back to Home</a></p>
  <p>Huraira Khurshid | Database Engine Developer</p>
</footer>