# Database Architecture — Study & Interview Notes

Source: [Databases In-Depth – Complete Course](https://www.youtube.com/watch?v=pPqazMTzNOM) (freeCodeCamp / Suraj Kumar, Ed Courses). Full transcript: `sqlite_internals_transcript.md`.

---

## 1. General Database Architecture (any DB, high level)

Client → **Network Layer** → **Front End** → **Execution Engine** → **Transaction Layer** → **Storage Engine** → **OS Interaction Layer** → Disk. Distribution components sit alongside for scale.

### Front End (query understanding)
| Component | Job |
|---|---|
| Tokenizer | Splits raw query string into tokens (keywords, identifiers, literals) |
| Parser | Checks tokens form a grammatically valid statement → produces a parse tree |
| Optimizer | Picks the cheapest execution plan (index scan vs full scan, join order, parallelism) |

Output of front end = bytecode / instruction list the backend can execute. Front end ≈ "compiler" for the query.

### Execution Engine (backend core)
- **Query Executor** — runs the plan/bytecode step by step.
- **Cache Manager** — avoids re-computation/re-fetch (LRU, LFU strategies).
- **Utility services** — auth, backup, metrics.

### Transaction Management
- **Transaction Manager** — enforces ACID (Atomicity, Consistency, Isolation, Durability), not all DBs guarantee all four fully, but atomicity is near-universal.
- **Lock Manager** — shared/exclusive locks, granularity varies: row, document, page, or whole-DB.
- **Concurrency Manager** — broader than locking; e.g. **MVCC** (Multi-Version Concurrency Control) is the common technique beyond plain locks.
- **Recovery Manager** — restores consistent state after a crash mid-transaction.

### Storage Engine (core of the DB — determines speed)
- **Disk Storage Manager** — organizes data on disk in fixed-size **pages** (commonly 4 KB).
- **Buffer Manager** — moves pages disk ↔ memory for processing (different from query cache — this is page-level I/O buffering).
- **Index Manager** — B-Tree / B+Tree / LSM-Tree management to minimize disk reads.

> Interview point: **Buffer Manager vs Cache Manager** — buffer manager deals with raw disk pages; cache manager deals with query/result-level caching (LRU/LFU) inside the execution engine. Commonly confused.

### OS / File System Interaction Layer
- Abstracts OS-specific system calls (`open`, `read`, `write`, `close`) so the DB is portable across Linux/Mac/Windows.

### Distributed Components (scale-out)
- **Shard Manager** — splits data across nodes.
- **Cluster Manager** — manages the set of servers as one logical unit.
- **Replication Manager** — copies data for availability (CAP theorem trade-off: consistency vs availability).

---

## 2. Disk I/O Fundamentals

- RAM: volatile, fast (~1–100 ns/op), expensive, limited size.
- Disk: persistent, slow (~1–100 ms/op), cheap, effectively unlimited.
- Disk = platter → **tracks** (concentric circles) × **sectors** (pie slices) → intersection = **file block/page** (typically 4 KB, can be 512 B–16 KB).
- **You can never read/write partial data — the entire page must move disk ↔ RAM to be processed**, even to touch one row.

### Why indexing matters (worked example, memorize the shape of this argument)
1 million records, 400 bytes/record, 4 KB pages → 10 records/page → **10⁵ pages** to scan linearly.
- No index: 10⁵ pages × 1 ms/IO ≈ **100 seconds**.
- 1-level index (10 bytes/entry → 400 entries/page → 250 pages): ≈ **250 ms**.
- 2-level index (250 entries fit in 1 page): ≈ **3 ms** (3 IOs total).
→ This is exactly **multi-level indexing = a B-Tree**.

### Data-structure comparison for 1M records (search/insert/delete, disk-IO cost)
| Structure | Search | Insert | Delete |
|---|---|---|---|
| Balanced BST | log₂(n) ≈ 20 IOs | 20 | 20 |
| Unbalanced BST (worst case, skewed) | n ≈ 10⁶ IOs | 10⁶ | 10⁶ |
| Sorted array | log₂(n) ≈ 20 (binary search) | 10⁶ (shifting) | 10⁶ (shifting) |
| **B-Tree (order 100)** | log₁₀₀(10⁶) ≈ **3 IOs** | 3 | 3 |

**Why B-Trees win:** cost of one disk fetch >> cost of extra in-memory comparisons. So pack as many keys as possible per node/page — you pay for the whole page anyway.

### B-Tree properties (order M)
- Max children per node = **M** → max keys per node = **M − 1**.
- Min keys per non-root node = **⌈M/2⌉**; root minimum = 1 key.
- All leaves at the same depth (self-balancing) → height = **O(logₘ n)**.
- Grows **upward** (toward root) on overflow/split — unlike BST which grows toward leaves. This is what makes multi-level indexing adapt automatically as data grows/shrinks.

### B-Tree vs B+Tree
| | B-Tree | B+Tree |
|---|---|---|
| Keys stored | Internal nodes + leaves | **Only leaves** (internal nodes just route) |
| Leaf linkage | No | Leaves linked as a list |
| Best for | General lookup | **Range queries** (sequential scan via leaf list) — why most RDBMS indexes use B+Trees |

---

## 3. SQLite Internal Architecture (concrete case study)

SQLite = lightweight, embedded, zero-config, serverless, ACID, crash-recoverable, thread-safe, cross-platform (~200–300 KB footprint). Contrast: MySQL/Postgres are client-server, built for high concurrency/scale; SQLite is for embedded/mobile/IoT (single process, no network layer needed).

### 7 components, 2 divisions

**Division 1 — "Front end" (SQL text → bytecode)**
1. **Tokenizer** (`tokenize.c`, fn `sqlite3RunParser`) — chars → tokens.
2. **Parser** (`parse.y`, uses the **Lemon** parser generator, SQLite's own — simpler than yacc/bison) — tokens → grammar rules → calls C functions (e.g. `sqlite3StartTable`).
3. **Code Generator** — emits bytecode (a compiled `sqlite3_stmt` object). Lifecycle: `sqlite3_prepare_v2` → bind params → `sqlite3_step` (run) → `sqlite3_reset` / `sqlite3_finalize` (destroy).

**Division 2 — "Back end" (bytecode → actual data ops)**
4. **VDBE — Virtual Database Engine / Virtual Machine** (`vdbe.c`, fn `sqlite3VdbeExec`) — a giant `switch` statement (~29,000+ lines) over opcodes (`OP_Goto`, `OP_Halt`, …), each with up to 3 operands. Still thinks in terms of tables/rows/columns. `sqlite3_stmt` and VDBE object are literally the same struct (type-cast between them).
5. **B-Tree layer** — one **separate B+Tree per table and per index**. VDBE never touches disk directly — only talks to B-Tree, which only operates on the **cached copy** of pages.
6. **Pager** (`pager.c`) — the single most important/heaviest component. Responsible for:
   - **Cache management** (page cache, **LRU** eviction, tracks "dirty pages" — a linked list of pages modified in memory but not yet flushed to disk).
   - **Transaction manager** — implements atomic commit/rollback.
   - **Lock manager** — locks the **entire DB file** (not row/table-level!) — exclusive write lock blocks all other readers/writers.
   - **Log manager** — rollback journal / write-ahead style logging.
7. **VFS — Virtual File System / OS Interface layer** — cross-platform wrapper over `open`/`read`/`write`/`close`.

**Call chain (memorize for interviews):**
`Client SQL → Tokenizer → Parser (Lemon) → Code Generator → sqlite3_stmt (VDBE) → VDBE exec → B-Tree → Pager → VFS → Disk`

### On-disk file format
- Entire DB = **one file** = an array of fixed-size **pages** (default 4 KB), numbered from 1.
- First 100 bytes = **DB header** (magic string "SQLite format 3", page size, format/read/write version, etc.).
- Every page has exactly one type: **B-Tree page** (table/index, interior/leaf), **free-list page**, **overflow page** (payload too big for one page), or **pointer-map page**.
- Special hidden table **`sqlite_master`** stores schema of all tables/indexes/views/triggers — checked on every `CREATE TABLE` to prevent name collisions. A single `CREATE TABLE` can trigger ~8–9 internal SQL statements (creating `sqlite_master` row, indexes, etc.).
- **Amalgamation build**: SQLite ships as one merged `sqlite3.c` + `sqlite3.h` (138+ source files combined) instead of separate compilation — recommended because it enables cross-file compiler optimizations (~5%+ perf gain) and is dead simple to embed.

### Two-Phase Commit (how SQLite guarantees Atomicity/Durability)
State involved: **Cache (RAM)** ↔ **Rollback Journal (disk)** ↔ **DB file (disk)**.

1. **Phase 1** (`sqlite3BtreeCommitPhaseOne` → `pagerCommitPhase1`):
   - Create the rollback journal (if not present) and write the **original/unmodified pages** into it — this is the "undo" record.
   - Flush journal to disk.
   - Write the modified (dirty) pages from cache into the actual DB file, flush to disk.
   - Journal **still exists on disk** at end of phase 1; locks still held.
2. **Phase 2** (`sqlite3BtreeCommitPhaseTwo` → `pagerCommitPhase2`):
   - Delete/truncate the journal (transaction now durably committed).
   - Release locks.

**Crash recovery:** if a crash happens between phase 1 and phase 2, journal still has the original pages → on restart SQLite detects journal/DB mismatch and **rolls back** DB file to the journal's original state. SQLite only supports **undo/rollback**, not redo, via this mechanism (classic rollback-journal mode; WAL mode is the newer alternative not covered in depth here).

### Locking states (SQLite locks whole file, not rows)
`UNLOCKED → SHARED → RESERVED → PENDING → EXCLUSIVE` (roughly) — one writer at a time DB-wide; write lock is exclusive (blocks all reads and writes).

---

## 4. Quick-fire Interview Answers

- **Why do databases use B-Trees over BSTs/arrays?** Disk I/O dominates cost; B-Trees minimize the *number of disk fetches* by maximizing keys-per-page/node, giving O(logₘ n) with small constant (e.g. 3 IOs for 1M rows vs 20 for a BST).
- **Buffer manager vs cache manager?** Buffer = raw page-level disk↔RAM management (storage engine). Cache = query/result caching in execution engine (LRU/LFU).
- **Lock manager vs concurrency manager?** Locking is one mechanism; concurrency control is the broader goal — MVCC is a common technique used alongside/instead of heavy locking.
- **How does a DB guarantee atomicity across a crash?** Write-ahead style logging: persist enough info (old values in a rollback journal, or new values in a WAL) *before* mutating the actual data file, so recovery can replay/undo to a consistent state.
- **B-Tree vs B+Tree, when to use which?** B+Tree for range-heavy workloads (linked leaves = fast sequential range scan); plain B-Tree can be marginally better for pure point lookups since data can live in internal nodes too.
- **Why is SQLite serverless/embeddable while Postgres/MySQL aren't (by default)?** Design target: SQLite = single-process, no network layer, minimal footprint, for local/mobile/IoT use; Postgres/MySQL = client-server, built for concurrent multi-client, high-throughput workloads.
- **What are the 7 SQLite components?** Tokenizer, Parser, Code Generator (front end) + VDBE, B-Tree, Pager, VFS (back end).
