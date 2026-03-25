# JunoDB & RocksDB — Why Disk Writes Are Fast Enough

## Why It's Still Fast Despite Disk Writes

### RocksDB's LSM-Tree Trick

Writes don't go to disk immediately — they land in RAM first:

```
Write request comes in
        │
        ▼
   MemTable (RAM)  ← write happens HERE first (super fast)
        │
        │  background thread flushes to disk
        ▼
     SST files (Disk/SSD)
```

- App gets acknowledgement **immediately** after writing to RAM
- Disk flush happens **asynchronously** in the background
- From the app's perspective, the write **feels like a RAM write**

---

## Sequential vs Random Writes

RocksDB only does **sequential writes** to disk — which are far faster than random writes:

```
Random writes (slow):    jump here → jump there → jump somewhere else
Sequential writes (fast): write → write → write → write (one straight line)
```

| Approach | Write Pattern | Speed |
|---|---|---|
| Traditional databases | Random writes | Slow on disk |
| RocksDB (LSM-Tree) | Sequential writes | Much faster |

---

## SSD vs HDD — Storage Speed Comparison

RocksDB is optimized specifically for **SSDs**:

| Storage | Speed |
|---|---|
| HDD (spinning disk) | ~100 MB/s |
| SSD | ~3,000 MB/s |
| RAM | ~50,000 MB/s |

> SSD is ~16x slower than RAM — but fast enough to hit **single-digit millisecond latency** at PayPal's scale.

---

## Redis vs JunoDB — The Real Tradeoff

| Factor | Redis | JunoDB |
|---|---|---|
| Write Speed | Faster (pure RAM) | Slightly slower |
| Data Safety | Risk of data loss | Persistent & safe |
| Data Size | Limited by RAM | Limited by disk (much larger) |
| Cost | Expensive (RAM is costly) | Cheaper (disk is cheap) |

---

## Bottom Line

> Disk is slower than RAM — but RocksDB's design minimizes that gap so much that JunoDB can still hit **single-digit millisecond latency** at **350 billion requests/day**.

PayPal didn't need Redis-level speed. They needed:
- ✅ Persistence
- ✅ Scale
- ✅ Consistency

**JunoDB delivers all three — even with disk writes.**
