---
layout: default
title: Home
---

<div class="hero">
  <h1>Huraira Khurshid</h1>
  <p>Database Engine Developer | PostgreSQL Internals | C Extensions | Systems Programming</p>
</div>

<div class="container">

<section>
<h2>👨‍💻 About Me</h2>

I'm a database engine developer with hands-on experience in PostgreSQL internals, C extension development, and diagnostic tooling architecture. Currently building production-grade PostgreSQL operational diagnostics at **Inzent**, with a focus on reliability, observability, and systems-level failure analysis.

<span class="status-badge">📍 Currently: Database Engine Developer at Inzent</span>

**What I Do:**
- Build PostgreSQL C extensions with executor hooks and memory context management
- Architect diagnostic collection pipelines with snapshot-consistent semantics
- Debug systems-level failures in database infrastructure
- Contribute to PostgreSQL ecosystem through open-source tooling

</section>

<section>
<h2>🔨 Current Work at Inzent</h2>

<div class="timeline">
  <div class="timeline-item">
    <h3>✅ Completed: pg_diag</h3>
    <p>Production-grade diagnostic collector with snapshot-consistent PostgreSQL internals capture across 50+ metrics. Implements REPEATABLE READ transaction semantics with graceful degradation and partial-success taxonomy.</p>
    <a href="projects/pg-diag.html" class="btn">View Deep Dive →</a>
  </div>
  
  <div class="timeline-item">
    <h3>🚧 In Progress: lasso_collector</h3>
    <p>EDB LASSO-inspired diagnostic pipeline with deterministic artifact generation, timeout handling, and CLI robustness for OS + database state collection.</p>
  </div>
  
  <div class="timeline-item">
    <h3>📋 Next: C-Based PostgreSQL Extensions</h3>
    <p>Moving to core engine observability with custom C extensions for performance instrumentation and failure analysis.</p>
  </div>
</div>

</section>

<section>
<h2>🚀 Featured Projects</h2>

<div class="project-grid">

<div class="project-card">
  <h3>pg_trace</h3>
  <div class="tech-stack">C, Executor Hooks, SPI, Memory Contexts</div>
  <p>Production-grade C extension implementing executor-level query instrumentation with safe memory management and recursive self-tracing prevention.</p>
  
  <div class="tags">
    <span class="tag">Executor Hooks</span>
    <span class="tag">Memory Contexts</span>
    <span class="tag">SPI Safety</span>
  </div>
  
  <a href="projects/pg-trace.html" class="btn">Deep Dive →</a>
  <a href="https://github.com/huraira213/pg_trace---PostgreSQL-Query-Tracing-Extension" class="btn btn-secondary">GitHub →</a>
</div>

<div class="project-card">
  <h3>pg_cext</h3>
  <div class="tech-stack">C, SPI, Varlena, Array API</div>
  <p>Comprehensive extension demonstrating PostgreSQL C API mastery including varlena handling, array operations, and SPI lifecycle management.</p>
  
  <div class="tags">
    <span class="tag">Varlena Internals</span>
    <span class="tag">Array Operations</span>
    <span class="tag">Type Safety</span>
  </div>
  
  <a href="projects/pg-cext.html" class="btn">Deep Dive →</a>
  <a href="https://github.com/huraira213/pg_cext" class="btn btn-secondary">GitHub →</a>
</div>

<div class="project-card">
  <h3>pg_diag</h3>
  <div class="tech-stack">Python, psycopg3, psutil</div>
  <p>Production-ready diagnostic tool with snapshot-consistent collection across 50+ PostgreSQL metrics. Built at Inzent for operational troubleshooting.</p>
  
  <div class="tags">
    <span class="tag">REPEATABLE READ</span>
    <span class="tag">Capability Detection</span>
    <span class="tag">Graceful Degradation</span>
  </div>
  
  <a href="projects/pg-diag.html" class="btn">Deep Dive →</a>
  <a href="https://github.com/huraira213/PG_Diag" class="btn btn-secondary">GitHub →</a>
</div>

<div class="project-card">
  <h3>PostgreSQL Query Profiler</h3>
  <div class="tech-stack">Python, pg_stat_statements</div>
  <p>Continuous query observability system converting cumulative PostgreSQL statistics into interval-based delta metrics with regression detection.</p>
  
  <div class="tags">
    <span class="tag">Delta Modeling</span>
    <span class="tag">Restart Detection</span>
    <span class="tag">Baseline Analysis</span>
  </div>
  
  <a href="projects/pg-query-profiler.html" class="btn">Deep Dive →</a>
  <a href="https://github.com/huraira213/PostgreSQL-Query-Profile" class="btn btn-secondary">GitHub →</a>
</div>

</div>

</section>

<section>
<h2>💡 What Makes My Work Different</h2>

**I don't just build tools - I understand what breaks them.**

Every project on this portfolio includes:
- **What PostgreSQL internals it demonstrates** (memory contexts, SPI, hooks, catalogs)
- **What broke during development** (and how I debugged it)
- **What production features are missing** (honest assessment)
- **What I'd do differently now** (showing growth)

This isn't tutorial-driven development - it's systems-level problem solving.

</section>

<section>
<h2>📚 Learning Path</h2>

- [PostgreSQL Internals](learning/postgresql-internals.html) - Resources for mastering PostgreSQL core
- [C Systems Programming](learning/c-systems-programming.html) - Foundation for database engine work
- [Currently Learning](learning/currently-learning.html) - What I'm studying at Inzent

</section>

<section>
<h2>📫 Get in Touch</h2>

**Email:** [hurairakhurshid4@gmail.com](mailto:hurairakhurshid4@gmail.com)  
**LinkedIn:** [linkedin.com/in/huraira-kiyani-05497429b](https://linkedin.com/in/huraira-kiyani-05497429b)  
**GitHub:** [github.com/huraira213](https://github.com/huraira213)

**Looking for:** PostgreSQL internals discussions, C extension collaboration, systems programming challenges

</section>

</div>