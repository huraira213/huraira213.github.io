---
layout: default
title: Resume - Huraira Khurshid
---

# Huraira Khurshid
**Database Engine Developer**

Rawalpindi, Pakistan | hurairakhurshid4@gmail.com  
[LinkedIn](https://linkedin.com/in/huraira-kiyani-05497429b) | [GitHub](https://github.com/huraira213) | [Portfolio](https://huraira213.github.io)

---

## Professional Summary

Database engine developer with hands-on PostgreSQL internals experience including C extension development, executor instrumentation, and diagnostic tooling architecture. Currently building PostgreSQL operational diagnostics at Inzent with focus on reliability, observability, and systems-level failure analysis. Strong foundation in memory context management, SPI integration, subprocess orchestration, and production-grade artifact generation.

---

## Technical Experience

### Database Engine Developer | Inzent | Feb 2026 – Present

**PostgreSQL Diagnostics Infrastructure:**
- Architected `pg_diag`: production-grade diagnostic collector with snapshot-consistent PostgreSQL internals capture across 50+ metrics (version info, settings, locks, replication, performance analysis)
- Implemented REPEATABLE READ transaction semantics for multi-query consistency with graceful degradation and partial-success taxonomy
- Built capability detection system for version-aware feature gating and extension availability checks
- Currently developing `lasso_collector`: EDB LASSO-inspired diagnostic pipeline with deterministic artifact generation, timeout handling, and CLI robustness for OS + database state collection

**Systems-Level Reliability Engineering:**
- Solved subprocess execution failure semantics with consistent artifact writing under timeout/error conditions
- Implemented secret redaction patterns for safe diagnostic sharing across untrusted boundaries
- Designed modular collector architecture enabling rapid extension addition without core pipeline changes
- Built CLI contract enforcement with `--dry-run`, `--skip-queries`, and flexible connection modes (Unix socket, TCP)

### Software Engineering Intern – Database Systems | SKAI Worldwide | May 2024 – Oct 2024

**PostgreSQL Extension Development:**
- Developed AgensAI extension implementing AI model inference within PostgreSQL using SPI and custom data type handling
- Migrated Apache AGE NetworkX driver to AgensGraph by adapting query translation logic while preserving graph operation semantics
- Reverse-engineered pgAdmin4 architecture to study extensibility patterns in large-scale database management systems
- Containerized PostgreSQL, AgensGraph, and Apache AGE development environments reducing setup time from hours to minutes

---

## Technical Projects

### pg_trace - PostgreSQL Query Tracing Extension | C, Executor Hooks, SPI
**Production-grade C extension implementing executor-level query instrumentation**

Demonstrates: Executor hooks, memory context lifecycle, SPI safety, recursive self-tracing prevention

- Implemented `ExecutorStart_hook` and `ExecutorEnd_hook` for query lifecycle capture with normalized query text generation
- Designed session-scoped memory contexts (`AllocSetContextCreate`) preventing backend memory leaks across query boundaries
- Solved recursive self-tracing through explicit query-text guards for extension catalog access
- Built slow-query threshold detection with configurable GUC parameters and set-returning function interfaces
- **Key Learning:** SPI writes from executor hook paths are unsafe; switched to in-memory queue with manual persistence

**GitHub:** [huraira213/pg_trace](https://github.com/huraira213/pg_trace---PostgreSQL-Query-Tracing-Extension)

---

### pg_cext - PostgreSQL Extension Development Foundation | C, SPI, Varlena
**Comprehensive extension demonstrating PostgreSQL C API mastery**

Demonstrates: Varlena internals, array operations, SPI lifecycle, error handling

- Implemented string manipulation with varlena header optimization (1-byte vs 4-byte header for short strings)
- Built array operations using `ARR_DATA_PTR` direct memory access patterns for O(n) performance
- Integrated SPI for dynamic query execution with proper `SPI_connect()` / `SPI_finish()` lifecycle management
- Demonstrated `palloc()` memory context awareness across function boundaries preventing memory corruption
- **Key Learning:** Memory allocated in PostgreSQL contexts persists beyond function scope; explicit cleanup required

**GitHub:** [huraira213/pg_cext](https://github.com/huraira213/pg_cext)

---

### pg_diag - PostgreSQL Diagnostic Collector | Python, psycopg3, psutil
**Production-ready diagnostic tool with snapshot-consistent collection**

Demonstrates: Transaction snapshot semantics, capability detection, graceful degradation, error taxonomy

- Architected `REPEATABLE READ, READ ONLY` snapshot semantics for multi-query consistency across system catalog views
- Implemented partial/error status taxonomy with embedded error detection for graceful degradation
- Built version-aware capability detection for extensions (`pg_stat_statements`), replication mode, and PostgreSQL feature flags
- Solved transaction rollback safety: explicit rollback logic prevents transaction poisoning after collector errors
- **Key Learning:** Cross-version `pg_stat_statements` columns differ; schema introspection + dynamic query generation required

**GitHub:** [huraira213/PG_Diag](https://github.com/huraira213/PG_Diag)

---

### PostgreSQL Query Profiler | Python, pg_stat_statements
**Continuous query observability with delta metrics and regression detection**

Demonstrates: Cumulative counter conversion, restart detection, delta modeling, baseline analysis

- Converted `pg_stat_statements` cumulative counters into interval-based delta metrics for time-window analysis
- Implemented restart detection preventing negative delta artifacts after PostgreSQL service restart
- Built cross-version compatibility via `information_schema.columns` introspection and dynamic SELECT generation
- Designed normalized schema separating snapshots, query definitions, raw metrics, and delta metrics
- **Key Learning:** Counter resets create negative deltas; single-pass restart detection with max() clamping required

**GitHub:** [huraira213/PostgreSQL-Query-Profile](https://github.com/huraira213/PostgreSQL-Query-Profile)

---

## Technical Skills

**Languages:** C, Python, SQL, Cypher  
**PostgreSQL Internals:** C Extension Development, SPI, Memory Contexts, Executor Hooks, Varlena Handling, System Catalogs, pg_stat_* Views, Query Optimization Basics  
**Database Systems:** PostgreSQL Core, AgensGraph, Apache AGE, Vector Databases  
**Systems Programming:** Memory Management, Process Models, Subprocess Orchestration, Error Handling, Debugging (GDB, Valgrind)  
**Tools & Infrastructure:** Docker, Git, psycopg2/3, psutil, PGXS Build System

---

## Education

**Bachelor of Science in Software Engineering**  
Virtual University of Pakistan 

**Relevant Coursework:** Database Systems, Operating Systems, Data Structures & Algorithms, Object-Oriented Programming, Software Architecture

---

## Community & Learning

- Active in PostgreSQL internals communities (r/PostgreSQL, pgsql-hackers mailing list)
- Studying PostgreSQL source code: executor (`src/backend/executor/`), memory contexts, SPI internals
- Contributing to open-source PostgreSQL ecosystem through diagnostic tooling and extension development

---

## References

Available upon request

---

## Download Markdown Resume

<div class="info-box">
  <p><strong>Download Source:</strong> <a href="about.md" download="Huraira_Khurshid_Resume.md">Download Resume (.md)</a></p>
</div>
