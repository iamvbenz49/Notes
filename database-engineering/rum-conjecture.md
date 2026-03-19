# The RUM Conjecture — Database Trade-offs

---

## What is the RUM Conjecture?

The RUM Conjecture states that any database system must trade off between three factors:

| Letter | Stands For | Meaning |
|---|---|---|
| **R** | Read | How fast data can be retrieved |
| **U** | Update | How fast data can be written/modified |
| **M** | Memory | How much storage space is used |

> **You can only optimize for two of the three. Improving two will always hurt the third.**

Visualize it as a triangle — you can only pick two sides to maximize.

```
          Read (R)
          /\
         /  \
        /    \
       /      \
Update (U) — Memory (M)
```

---

## 1. Read-Optimized Workloads

Prioritize **fast data retrieval** at the cost of higher storage and slower writes.

- **B-Trees** — Used in most relational databases. Offer O(log N) lookups but require maintaining auxiliary nodes, which takes up space and makes updates slower.
- **Tries** — Tree structure optimized for prefix-based lookups.
- **Skip Lists** — Layered linked lists that allow fast search at the cost of extra memory.

**Trade-off:** Fast reads ✅ | Slow updates ❌ | High memory usage ❌

---

## 2. Update (Write) Optimized Workloads

Prioritize **fast write/ingestion speed**, typically by appending data rather than modifying in place.

- **Log-Structured Storage** — New records are simply appended to a file. Writes are very fast, but reads require scanning the file linearly.
- **LSM Trees (Log-Structured Merge-Trees)** — Accept writes in memory and flush to disk periodically. Great write latency, but require an additional in-memory buffer.

**Trade-off:** Fast updates ✅ | Slow reads ❌ | Extra memory needed ❌

---

## 3. Space (Memory) Optimized Workloads

Prioritize **minimizing storage footprint**, usually sacrificing some read performance or accuracy.

- **Bloom Filters** — Highly space-efficient probabilistic structures, but introduce the risk of **false positives** (may say data exists when it doesn't).
- **Sparse Index** — Stores fewer index entries to save space, but reads become costlier because extra lookups are needed to locate the actual data.

**Trade-off:** Low memory ✅ | Slower/less accurate reads ❌ | Slower updates ❌

---

## 4. Adaptive Access Methods

These sit **in the middle of the RUM triangle** and adjust dynamically based on current workload patterns rather than being locked into one corner.

### The Problem They Solve

With B-Trees, LSM Trees, Bloom Filters etc., you **commit upfront** to a trade-off at design time. But what if your workload changes over time? Morning might be write-heavy (lots of inserts), evening might be read-heavy (lots of queries). A fixed structure can't adapt — it just suffers during the wrong workload.

Adaptive access methods solve this by **reorganizing themselves based on actual usage patterns**.

### Database Cracking

The most well-known adaptive method.

> **Every query teaches the database something. Use that knowledge to reorganize data.**

Say you have an unsorted column of 1 million values and you query:

```sql
SELECT * FROM table WHERE age BETWEEN 20 AND 40;
```

A normal database scans everything. But a cracking database also **physically rearranges** the data as a side effect of answering the query — splitting it into partitions:

```
[ values < 20 ] | [ values 20–40 ] | [ values > 40 ]
```

The next time someone queries in that range, the search space is already smaller. Each subsequent query **cracks** the data further into finer partitions. Over time, without any explicit indexing step, the data becomes progressively more organized around the actual queries being asked.

- No upfront cost to build an index
- The index builds itself incrementally through normal query traffic
- Cold start is slow, gets faster over time

### Adaptive Indexing

A broader concept — the system **monitors query patterns** and automatically decides when and where to build, merge, or drop indexes.

- If a column is queried frequently → build an index on it
- If an index hasn't been used in a while → drop it to save space
- If write load spikes → temporarily relax index maintenance

### Tunable Parameters

Many modern databases expose knobs that let you **slide along the RUM triangle** rather than being stuck at one corner.

- **LSM Tree compaction aggressiveness** — compact more often (better reads, more I/O) vs. less often (faster writes, more space used)
- **Buffer pool size** — larger buffer = faster reads, more memory used
- **Index fill factor** — how full each B-Tree page is kept, trading read vs. write performance

### Where They Sit on the RUM Triangle

```
          Read (R)
          /\
         /  \
        / 🎯 \    ← Adaptive methods live here
       /      \       in the middle, shifting as needed
Update (U) — Memory (M)
```

They don't fully win any corner — but they don't fully lose any either. The goal is **good enough at everything** rather than **perfect at one thing**.

### Trade-offs

| Pro | Con |
|---|---|
| No upfront design commitment | Complexity — harder to reason about |
| Self-optimizes over time | Unpredictable performance early on (cold start) |
| Handles mixed/changing workloads | Reorganization has its own CPU/memory cost |
| No manual DBA tuning needed | Can make wrong assumptions about future patterns |

### Real World Usage

- **MonetDB** — pioneered database cracking
- **CockroachDB / PostgreSQL** — adaptive statistics and partial auto-indexing
- **RocksDB** — tunable LSM compaction strategies
- **Amazon Aurora** — auto-tunes based on workload automatically

---

## Using the RUM Framework

When analyzing any database system or data structure, ask:

1. Which two factors is it optimizing for?
2. Which factor is it sacrificing?
3. Does the workload match the trade-off?

This framework applies to any storage system — not just databases.
