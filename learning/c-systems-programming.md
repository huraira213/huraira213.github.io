---
layout: default
title: C Systems Programming - Learning Resources
---

<div class="page-header">
  <h1>C Systems Programming</h1>
  <p class="tagline">Foundation for Database Engine Work</p>
</div>

<div class="container">

<div class="terminal-section">
  <div class="terminal-header">
    <span class="terminal-title">$ gcc --version</span>
  </div>
  <div class="terminal-body">
    <p>Systems programming in C is the foundation for database engine development. This guide covers essential C concepts, debugging tools, and best practices for building reliable low-level software.</p>
  </div>
</div>

<section>
<h2>Essential Books</h2>

### 1. The C Programming Language (K&R)
**Authors:** Brian Kernighan, Dennis Ritchie  
**Status:** Completed

**What I Learned:**
- Core C syntax and semantics
- Pointers and memory management
- Standard library fundamentals

**Best For:** Foundation - read this first

---

### 2. Expert C Programming (Peter van der Linden)
**Status:** Reading in progress  
**Focus:** Deep C knowledge and gotchas

**Key Topics:**
- How the linker works
- Arrays vs pointers (they're different!)
- Type declarations demystified
- C runtime internals

**Favorite Quote:** "C is quirky, flawed, and an enormous success"

---

### 3. Advanced Programming in the UNIX Environment (Stevens)
**Status:** Reference material  
**Focus:** Systems programming patterns

**Essential Chapters:**
- File I/O (Chapter 3)
- Process environment (Chapter 7)
- Signals (Chapter 10)
- Threads (Chapter 11)

---

### 4. Computer Systems: A Programmer's Perspective (Bryant/O'Hallaron)
**Status:** Ongoing study  
**Focus:** Low-level understanding

**Key Sections:**
- Memory hierarchy
- Virtual memory
- System-level I/O
- Network programming

</section>

<section>
<h2>Core Concepts for Database Work</h2>

### 1. Memory Management

**The Golden Rule:** In long-running processes (like databases), every `malloc` must have a matching `free`.
```c
// WRONG - Memory leak in database backend
void process_query(const char *sql) {
    char *normalized = malloc(strlen(sql) + 1);
    strcpy(normalized, sql);
    normalize_whitespace(normalized);
    execute(normalized);
    // Forgot to free! Leaks on every query.
}

// RIGHT - Proper cleanup
void process_query(const char *sql) {
    char *normalized = malloc(strlen(sql) + 1);
    if (!normalized) {
        elog(ERROR, "out of memory");
        return;
    }
    
    strcpy(normalized, sql);
    normalize_whitespace(normalized);
    execute(normalized);
    
    free(normalized);  // Always free!
}
```

**Why This Matters:**
- Database backends run for days/weeks
- Small leaks accumulate: 100 bytes/query × 1M queries = 100GB
- OOM killer terminates your process

---

### 2. Pointer Arithmetic

**Arrays and pointers are NOT the same:**
```c
char str[] = "hello";  // Array: sizeof(str) = 6
char *ptr = "hello";   // Pointer: sizeof(ptr) = 8 (on 64-bit)

// Array to pointer decay
void process(char *s) {
    // sizeof(s) is ALWAYS pointer size, not string length!
}
```

**Pointer arithmetic for performance:**
```c
// Slow - repeated indexing
double sum_slow(double *arr, int n) {
    double total = 0;
    for (int i = 0; i < n; i++) {
        total += arr[i];  // arr + i computed every iteration
    }
    return total;
}

// Fast - pointer increment
double sum_fast(double *arr, int n) {
    double total = 0;
    double *end = arr + n;
    for (double *p = arr; p < end; p++) {
        total += *p;  // Direct dereference, no arithmetic
    }
    return total;
}
```

**PostgreSQL uses this everywhere** (e.g., `ARR_DATA_PTR` for arrays)

---

### 3. Struct Alignment and Padding
```c
// Naively might expect sizeof = 9
struct Bad {
    char a;     // 1 byte
    int b;      // 4 bytes
    char c;     // 1 byte
};
// Actual sizeof = 12 (padding inserted!)

// Better layout - minimize padding
struct Good {
    int b;      // 4 bytes
    char a;     // 1 byte
    char c;     // 1 byte
    // 2 bytes padding
};
// sizeof = 8
```

**Why This Matters:**
- Database rows are packed structs
- Wasted padding = wasted disk space
- Alignment affects performance (unaligned access is slower)

---

### 4. Undefined Behavior

**Most dangerous C pitfalls:**
```c
// 1. Using uninitialized memory
int x;
if (x == 5) { ... }  // UB: x has garbage value

// 2. Buffer overflow
char buf[10];
strcpy(buf, "this string is way too long");  // UB: overflow

// 3. Use after free
char *p = malloc(100);
free(p);
strcpy(p, "hello");  // UB: p is dangling

// 4. Null pointer dereference
char *p = NULL;
*p = 'a';  // UB: segfault

// 5. Signed integer overflow
int x = INT_MAX;
x = x + 1;  // UB: signed overflow is undefined!
```

**Defense:**
- Initialize all variables
- Use safe string functions (`strncpy`, `snprintf`)
- NULL pointers after free
- Check all return values
- Use unsigned types when overflow should wrap

</section>

<section>
<h2>Debugging Tools</h2>

### 1. GDB (GNU Debugger)

**Essential commands:**
```bash
# Compile with debug symbols
gcc -g -O0 myprogram.c -o myprogram

# Start debugging
gdb ./myprogram

# GDB commands
(gdb) break main           # Set breakpoint at main
(gdb) run arg1 arg2        # Run program with arguments
(gdb) next                 # Step over
(gdb) step                 # Step into
(gdb) print variable       # Print variable value
(gdb) backtrace            # Show call stack
(gdb) info locals          # Show local variables
(gdb) watch var            # Break when var changes
(gdb) continue             # Continue execution
```

**PostgreSQL-specific:**
```bash
# Attach to running backend
gdb -p $(pidof postgres | awk '{print $1}')

# Set breakpoint in extension
(gdb) break my_extension.c:42
(gdb) continue
```

---

### 2. Valgrind (Memory Error Detector)

**Detect memory leaks:**
```bash
# Run under Valgrind
valgrind --leak-check=full ./myprogram

# Output shows:
# - Invalid reads/writes
# - Memory leaks
# - Use after free
```

**Example output:**
```
==12345== Invalid write of size 4
==12345==    at 0x40052E: bad_function (bad.c:10)
==12345==  Address 0x0 is not stack'd, malloc'd or free'd

==12345== LEAK SUMMARY:
==12345==    definitely lost: 100 bytes in 1 blocks
```

**For PostgreSQL extensions:**
```bash
valgrind --leak-check=full \
         --suppressions=/path/to/postgres.supp \
         postgres -D /path/to/data
```

---

### 3. AddressSanitizer (Faster than Valgrind)
```bash
# Compile with sanitizer
gcc -g -fsanitize=address myprogram.c -o myprogram

# Run normally - crashes on error
./myprogram

# Shows exact line of memory error
```

**Catches:**
- Buffer overflows
- Use after free
- Use after return
- Double free

---

### 4. Static Analysis Tools

**Cppcheck:**
```bash
cppcheck --enable=all mycode.c
```

**Clang Static Analyzer:**
```bash
clang --analyze mycode.c
```

</section>

<section>
<h2>Practice Projects</h2>

### Level 1: Memory Management
**Goal:** Master dynamic allocation
```c
// Build a dynamic string buffer
typedef struct {
    char *data;
    size_t len;
    size_t capacity;
} StringBuffer;

StringBuffer *sb_create();
void sb_append(StringBuffer *sb, const char *str);
void sb_destroy(StringBuffer *sb);
```

**Learn:** Reallocation strategy, avoiding fragmentation

---

### Level 2: Data Structures
**Goal:** Implement core structures
```c
// Hash table
typedef struct HashTable HashTable;
HashTable *ht_create(size_t size);
void ht_insert(HashTable *ht, const char *key, void *value);
void *ht_get(HashTable *ht, const char *key);
void ht_destroy(HashTable *ht);
```

**Learn:** Pointer manipulation, collision handling

---

### Level 3: File I/O
**Goal:** Understand system calls
```c
// Simple database: append-only log
int db_open(const char *filename);
int db_append(int fd, const char *key, const char *value);
char *db_get(int fd, const char *key);
void db_close(int fd);
```

**Learn:** `open`, `read`, `write`, `lseek`, buffering

---

### Level 4: Process Management
**Goal:** Multi-process coordination
```c
// Fork-based worker pool
void worker_pool_create(int num_workers);
void worker_pool_submit(void (*task)(void *), void *arg);
void worker_pool_destroy();
```

**Learn:** `fork`, `exec`, `wait`, pipes, signals

</section>

<section>
<h2>Common Patterns in PostgreSQL Code</h2>

### 1. Error Handling with `elog`
```c
// Instead of returning error codes
if (!SPI_connect()) {
    elog(ERROR, "SPI_connect failed");
    // Never reaches here - ERROR throws exception
}

// Levels: DEBUG, LOG, INFO, NOTICE, WARNING, ERROR, FATAL
elog(DEBUG1, "variable = %d", my_var);
```

---

### 2. Memory Context Pattern
```c
// Create scoped context
MemoryContext oldcontext = MemoryContextSwitchTo(MyContext);

// All allocations go to MyContext
char *data = palloc(1024);

// Switch back
MemoryContextSwitchTo(oldcontext);

// Later: destroy entire context (frees all allocations)
MemoryContextDelete(MyContext);
```

---

### 3. List Manipulation
```c
// PostgreSQL uses doubly-linked lists extensively
List *mylist = NIL;
mylist = lappend(mylist, item1);
mylist = lappend(mylist, item2);

// Iterate
ListCell *cell;
foreach(cell, mylist) {
    MyStruct *item = (MyStruct *) lfirst(cell);
    process(item);
}
```

</section>

<section>
<h2>Online Resources</h2>

**Interactive Learning:**
- [Learn C](https://www.learn-c.org/) - Interactive tutorials
- [C Programming Exercises](https://www.w3resource.com/c-programming-exercises/)

**Reference:**
- [C Standard Library Reference](https://en.cppreference.com/w/c)
- [POSIX Specification](https://pubs.opengroup.org/onlinepubs/9699919799/)

**Videos:**
- [CS50 Harvard (C lectures)](https://cs50.harvard.edu/)
- [Jacob Sorber YouTube](https://www.youtube.com/c/JacobSorber) - Systems programming

**Practice:**
- [Exercism C Track](https://exercism.org/tracks/c)
- [LeetCode (filter by C)](https://leetcode.com/)

</section>

<section>
<h2>Quick Reference</h2>

**Compilation:**
```bash
# Debug build
gcc -g -O0 -Wall -Wextra program.c -o program

# Release build
gcc -O3 -DNDEBUG program.c -o program

# With libraries
gcc program.c -o program -lpthread -lm

# Generate assembly
gcc -S program.c  # Creates program.s
```

**Common Flags:**
- `-g` - Debug symbols
- `-O0` - No optimization (for debugging)
- `-O3` - Maximum optimization
- `-Wall -Wextra` - Enable warnings
- `-Werror` - Treat warnings as errors
- `-std=c11` - Use C11 standard

**Memory Management:**
```c
malloc()   // Allocate uninitialized memory
calloc()   // Allocate zero-initialized memory
realloc()  // Resize allocation
free()     // Deallocate

// ALWAYS check malloc return:
ptr = malloc(size);
if (!ptr) {
    perror("malloc failed");
    exit(1);
}
```

</section>

</div>

<footer>
  <p><a href="../index.html">← Back to Home</a></p>
  <p>Huraira Khurshid | Database Engine Developer</p>
</footer>