---
layout: default
title: PostgreSQL Internals - Learning Resources
---

<div class="page-header">
  <h1>PostgreSQL Internals</h1>
  <p class="tagline">Resources for Mastering PostgreSQL Core</p>
</div>

<div class="container">

<div class="terminal-section">
  <div class="terminal-header">
    <span class="terminal-title">$ psql -c "SELECT version();"</span>
  </div>
  <div class="terminal-body">
    <p>PostgreSQL internals study path for aspiring database engine developers and PostgreSQL Global Development Group contributors.</p>
  </div>
</div>

<section>
<h2>Essential Books</h2>

### 1. PostgreSQL 14 Internals (Egor Rogov)
**Status:** Reading in progress  
**Focus:** Deep dive into PostgreSQL architecture

**Key Chapters:**
- Part I: Isolation and MVCC
- Part II: Buffer Cache and WAL
- Part III: Locks and Query Execution
- Part IV: Query Planning and Optimization

**Why Read This:**
- Written by PostgreSQL contributor
- Covers core concepts with source code references
- Explains MVCC implementation details
- Available in English (translated from Russian)

**Link:** [postgrespro.com/community/books/internals](https://postgrespro.com/community/books/internals)

---

### 2. PostgreSQL Server Programming (Hannu Krosing, Jim Mlodgenski)
**Status:** Completed  
**Focus:** Extension development, PL/pgSQL, C functions

**What I Learned:**
- Server Programming Interface (SPI) patterns
- Custom data type creation
- Background worker architecture
- Hook system fundamentals

**Best For:** Building your first PostgreSQL extension

---

### 3. The Internals of PostgreSQL (Hironobu Suzuki)
**Status:** Reference material  
**Focus:** Architecture diagrams and process flows

**Online Resource:** [interdb.jp/pg](http://www.interdb.jp/pg/)

**Covers:**
- Backend process architecture
- Memory architecture
- Query processing pipeline
- Vacuum and autovacuum internals

</section>

<section>
<h2>Official Documentation (Must-Read)</h2>

### Core Documentation Sections

**1. Server Programming Interface (SPI)**  
[postgresql.org/docs/current/spi.html](https://www.postgresql.org/docs/current/spi.html)

Read when:
- Building C extensions that execute SQL
- Understanding transaction boundaries in functions
- Learning memory context management

---

**2. C Language Functions**  
[postgresql.org/docs/current/xfunc-c.html](https://www.postgresql.org/docs/current/xfunc-c.html)

Covers:
- Function manager interface
- Argument/return handling (`PG_GETARG_*`, `PG_RETURN_*`)
- NULL handling (`PG_ARGISNULL`)
- Memory contexts (`palloc`, `pfree`)

---

**3. Writing PostgreSQL Extensions**  
[postgresql.org/docs/current/extend.html](https://www.postgresql.org/docs/current/extend.html)

Essential for:
- Extension packaging (`.control`, `--version.sql`)
- PGXS build system
- Extension dependencies

---

**4. System Catalogs**  
[postgresql.org/docs/current/catalogs.html](https://www.postgresql.org/docs/current/catalogs.html)

Critical tables:
- `pg_stat_activity` - Current session activity
- `pg_stat_database` - Database-wide statistics
- `pg_stat_statements` - Query performance metrics
- `pg_locks` - Lock information
- `pg_class` - Tables and indexes

---

**5. Monitoring Stats Views**  
[postgresql.org/docs/current/monitoring-stats.html](https://www.postgresql.org/docs/current/monitoring-stats.html)

Understanding:
- Cumulative vs snapshot statistics
- Statistics collector architecture
- `pg_stat_*` view internals

</section>

<section>
<h2>Source Code Study Path</h2>

**Repository:** [git.postgresql.org/git/postgresql.git](https://git.postgresql.org/git/postgresql.git)

### Recommended Reading Order

**Phase 1: Extension Basics**
```bash
# Study these files first
src/backend/utils/fmgr/fmgr.c          # Function manager
src/backend/executor/spi.c             # SPI implementation
src/include/fmgr.h                     # Function macros
contrib/pg_stat_statements/            # Real-world extension
```

**Phase 2: Memory Management**
```bash
src/backend/utils/mmgr/mcxt.c          # Memory contexts
src/backend/utils/mmgr/aset.c          # AllocSet implementation
src/include/utils/palloc.h             # Memory allocation
```

**Phase 3: Data Types**
```bash
src/backend/utils/adt/varlena.c        # Variable-length types
src/backend/utils/adt/arrayfuncs.c     # Array operations
src/backend/utils/adt/numeric.c        # Numeric type
```

**Phase 4: Executor**
```bash
src/backend/executor/execMain.c        # Executor main loop
src/backend/executor/nodeSeqscan.c     # Sequential scan
src/include/executor/executor.h        # Executor hooks
```

### How to Study Source Code

**1. Build PostgreSQL from source:**
```bash
git clone https://git.postgresql.org/git/postgresql.git
cd postgresql
./configure --enable-debug --enable-cassert CFLAGS="-ggdb -O0"
make -j$(nproc)
sudo make install
```

**2. Use code search tools:**
- [doxygen.postgresql.org](https://doxygen.postgresql.org/)
- GitHub code search
- `grep -r "function_name" src/`

**3. Add debug logging:**
```c
elog(NOTICE, "Debug: variable = %d", my_var);
```

</section>

<section>
<h2>Online Courses & Tutorials</h2>

### Free Resources

**1. Postgres Professional Training**  
[postgrespro.com/education](https://postgrespro.com/education)

Courses:
- PostgreSQL Internals
- Query Optimization
- Administration

---

**2. Bruce Momjian's Presentations**  
[momjian.us/main/presentations](https://momjian.us/main/presentations/)

Watch:
- "Inside PostgreSQL Shared Memory"
- "MVCC Unmasked"
- "Explaining the Postgres Query Optimizer"

---

**3. Cybertec PostgreSQL Blog**  
[cybertec-postgresql.com/en/blog](https://www.cybertec-postgresql.com/en/blog/)

Technical deep dives on:
- MVCC internals
- Query optimization tricks
- Extension development patterns

</section>

<section>
<h2>Practice Projects (Recommended Order)</h2>

### Level 1: Basic C Extension
**Goal:** Understand function manager interface
```c
// Simple arithmetic functions
Datum add_nums(PG_FUNCTION_ARGS);
Datum factorial(PG_FUNCTION_ARGS);
```

**Learn:** `PG_FUNCTION_INFO_V1`, `PG_GETARG_*`, `PG_RETURN_*`

---

### Level 2: Varlena Handling
**Goal:** Master variable-length types
```c
// String manipulation
Datum reverse_string(PG_FUNCTION_ARGS);
Datum concat_text(PG_FUNCTION_ARGS);
```

**Learn:** `VARSIZE_ANY_EXHDR`, `SET_VARSIZE`, `VARDATA_ANY`

---

### Level 3: SPI Integration
**Goal:** Execute SQL from C
```c
// Dynamic query execution
Datum get_table_count(PG_FUNCTION_ARGS);
Datum execute_query(PG_FUNCTION_ARGS);
```

**Learn:** `SPI_connect`, `SPI_execute`, `SPI_finish`

---

### Level 4: Executor Hooks
**Goal:** Intercept query execution
```c
// Query instrumentation
ExecutorStart_hook_type prev_ExecutorStart;
static void my_executor_start(QueryDesc *queryDesc, int eflags);
```

**Learn:** Hook registration, query lifecycle, memory contexts

---

### Level 5: Background Workers
**Goal:** Long-running processes
```c
// Async processing
void my_worker_main(Datum main_arg);
```

**Learn:** Worker registration, shared memory, IPC

</section>

<section>
<h2>Mailing List & Community</h2>

### Essential Mailing Lists

**1. pgsql-hackers**  
[postgresql.org/list/pgsql-hackers](https://www.postgresql.org/list/pgsql-hackers/)

**What to learn:**
- Follow patch discussions
- See how features are designed
- Understand code review process

**How to start:**
- Subscribe (digest mode recommended)
- Read threads on topics you're learning
- Don't post until you've lurked for months

---

**2. pgsql-performance**  
[postgresql.org/list/pgsql-performance](https://www.postgresql.org/list/pgsql-performance/)

**Focus:**
- Query optimization discussions
- `EXPLAIN` plan analysis
- Performance troubleshooting patterns

---

**3. Reddit: r/PostgreSQL**  
[reddit.com/r/PostgreSQL](https://www.reddit.com/r/PostgreSQL/)

Good for:
- Quick questions
- Extension recommendations
- Community discussions

</section>

<section>
<h2>Learning Milestones</h2>

<div class="timeline">
  <div class="timeline-item">
    <h3>Milestone 1: First Extension</h3>
    <p>Build and install a basic C extension with arithmetic functions</p>
    <p><strong>Validation:</strong> Can explain `PG_FUNCTION_INFO_V1` macro</p>
  </div>
  
  <div class="timeline-item">
    <h3>Milestone 2: Memory Context Mastery</h3>
    <p>Build extension with proper `palloc` usage and no leaks</p>
    <p><strong>Validation:</strong> Run under Valgrind with zero leaks</p>
  </div>
  
  <div class="timeline-item">
    <h3>Milestone 3: SPI Integration</h3>
    <p>Execute dynamic SQL from C function safely</p>
    <p><strong>Validation:</strong> Handle SPI errors gracefully</p>
  </div>
  
  <div class="timeline-item">
    <h3>Milestone 4: Executor Hooks (Current)</h3>
    <p>Intercept query execution with custom hook</p>
    <p><strong>Validation:</strong> Trace queries without crashing</p>
  </div>
  
  <div class="timeline-item">
    <h3>Milestone 5: First Core Patch</h3>
    <p>Submit patch to pgsql-hackers (even if rejected)</p>
    <p><strong>Validation:</strong> Get feedback from committers</p>
  </div>
</div>

</section>

<section>
<h2>Quick Reference Commands</h2>
```bash
# Build extension
make clean && make && sudo make install

# Reload in PostgreSQL
DROP EXTENSION IF EXISTS my_ext;
CREATE EXTENSION my_ext;

# Check for memory leaks
valgrind --leak-check=full postgres -D /path/to/data

# Debug with GDB
gdb --args postgres -D /path/to/data
(gdb) break my_function
(gdb) run

# View system catalogs
psql -c "\d pg_stat_statements"
psql -c "SELECT * FROM pg_extension;"

# Monitor backend processes
ps aux | grep postgres
```

</section>

<section>
<h2>Additional Resources</h2>

**Tools:**
- [pgAdmin4](https://www.pgadmin.org/) - GUI for PostgreSQL
- [pg_top](https://pg_top.gitlab.io/) - Real-time monitoring
- [pgBadger](https://pgbadger.darold.net/) - Log analyzer

**Extensions to Study:**
- `pg_stat_statements` - Query performance tracking
- `pg_trgm` - Trigram similarity search
- `pgcrypto` - Cryptographic functions
- `hstore` - Key-value storage

**Blogs:**
- [2ndQuadrant Blog](https://www.2ndquadrant.com/en/blog/)
- [Crunchy Data Blog](https://www.crunchydata.com/blog)
- [EnterpriseDB Blog](https://www.enterprisedb.com/blog)

</section>

</div>

<footer>
  <p><a href="../index.html">← Back to Home</a></p>
  <p>Huraira Khurshid | Database Engine Developer</p>
</footer>