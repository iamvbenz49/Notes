# Time Series Database Compression — Notes

---

## The Problem

Storing large streams of metrics naively is expensive.

- Hardware metrics like CPU utilization, memory usage, and API latency generate millions of data points.
- Naive approach: 1 million 64-bit integers = **8 bytes × 1,000,000 = 8MB**

The key insight that makes compression possible: **time series data has low variance** — consecutive values are usually very close to each other.

---

## Technique 1: Delta Encoding

Instead of storing the actual values, store only the **difference (delta)** between consecutive numbers.

### Example

```
Raw:    10000, 10005, 10007, 10009
Deltas: 10000,     5,     2,     2
```

The numbers go from 5-digit integers to single digits — far cheaper to store.

### Trade-off

| | Detail |
|---|---|
| **Gain** | Much smaller numbers → less storage space |
| **Loss** | CPU cost on read — must reconstruct originals by summing deltas from the base value |

Works best for metrics with **low variance** (steady state) — CPU usage, temperature sensors, counters.

---

## Technique 2: Variable Length Integer Encoding (VLE)

Fixed-size types always use 8 bytes, even for tiny numbers like `2` or `5`. VLE fixes this by using **only as many bytes as the number actually needs**.

### How It Works — Continuation Bits

Each byte uses 1 bit to signal whether more bytes follow:

- `0` in the leading bit → this is the last byte for this number
- `1` in the leading bit → more bytes follow

### Example

The number `292` in binary is `100100100` — too big for 1 byte (max 127), but fits in 2 bytes using VLE instead of the usual 8.

```
Raw fixed:  8 bytes for every number
VLE:        1 byte for 0–127
            2 bytes for 128–16,383
            ... and so on
```

---

## Combining Both: Maximum Compression

Delta encoding produces small numbers → VLE then stores those small numbers with minimal bytes.

### Benchmarks (1 Million Data Points)

| Method | Storage Size |
|---|---|
| Raw Data | 8 MB |
| Variable Length Encoding only | 3 MB |
| Variable Length + Delta Encoding | **1 MB** |

**8× reduction** from combining both techniques.

---

## Advanced: Delta-of-Delta Encoding

If the deltas themselves are still large numbers, apply delta encoding **again** on the deltas.

> **Delta** = how much the value changed.
> **Delta-of-delta** = how much the *change itself* changed.

### Step by Step

Start with:
```
Raw:  100, 110, 120, 130, 145
```

**Step 1 — Find the gaps (Delta):** subtract each number from the next:
```
110 - 100 = 10
120 - 110 = 10
130 - 120 = 10
145 - 130 = 15

Deltas: 10, 10, 10, 15
```

**Step 2 — Find the gaps of the gaps (Delta-of-delta):** subtract each delta from the next:
```
10 - 10 = 0
10 - 10 = 0
15 - 10 = 5

Delta-of-deltas: 0, 0, 5
```

- `0` because the gap **didn't change** — it was 10, then 10 again.
- `5` because the gap **jumped** — it was 10, then suddenly 15.

When intervals are perfectly regular, all delta-of-deltas = 0 → encodes to **1 bit each**.

### Real Timestamp Example (Gorilla / Prometheus style)

```
Raw timestamps (ms):     1710000000000, 1710000010000, 1710000020000, 1710000030000
After delta:                             10000,          10000,          10000
After delta-of-delta:                        0,              0,              0
```

Even with small jitter:
```
Raw timestamps:    ..., 1710000030000, 1710000045000  ← slight jitter
Deltas:            ..., 10000, 15000
Delta-of-delta:    ..., 0, +5000
```

- `0` → **1 bit**
- `+5000` → a few bytes, but this is rare

Result: thousands of regular timestamps often average **less than 1 byte each**.

---

## Decoding Delta-of-Delta

Decoding is the exact reverse — undo each step in reverse order.

**Encoding was:** Raw → Delta → Delta-of-delta

**Decoding is:** Delta-of-delta → Delta → Raw

The encoder always stores the **first value and first delta** as raw numbers, then stores delta-of-deltas from the third point onward:

```
Stored:  100 (first value),  10 (first delta),  0, 0, +5 (DoDs)
```

**Step 1 — Recover deltas** (cumulative sum of DoDs):
```
Delta[1] = 10
Delta[2] = 10 + 0 = 10
Delta[3] = 10 + 0 = 10
Delta[4] = 10 + 5 = 15
```

**Step 2 — Recover raw values** (cumulative sum of deltas):
```
Value[1] = 100
Value[2] = 100 + 10 = 110
Value[3] = 110 + 10 = 120
Value[4] = 120 + 10 = 130
Value[5] = 130 + 15 = 145
```

### In Code

```python
def decode(first_value, first_delta, dods):
    # Step 1: recover deltas
    deltas = [first_delta]
    for dod in dods:
        deltas.append(deltas[-1] + dod)

    # Step 2: recover raw values
    values = [first_value]
    for d in deltas:
        values.append(values[-1] + d)

    return values

first_value = 100
first_delta = 10
dods = [0, 0, +5]

print(decode(first_value, first_delta, dods))
# → [100, 110, 120, 130, 145]
```

Both encoding and decoding are just **running sums** — extremely cheap operations. This is why delta-of-delta is used in production: great compression AND fast decompression.

The same logic applies to values — if temperature stays at 23.4°C, all deltas = 0 → 1 bit each.

---

## How VLE Encodes Delta-of-Delta (Gorilla Style)

| Delta-of-Delta Value | Encoding | Bits Used |
|---|---|---|
| `0` | `0` | 1 bit |
| `-63` to `64` | `10` + 7 bits | 9 bits |
| `-255` to `256` | `110` + 9 bits | 12 bits |
| Large outlier | `111` + up to 32 bits | 35 bits |

~96% of timestamps in regular series hit the **1-bit case**.

---

## Why This Works So Well

- **Skewed distribution** — most deltas are tiny, so short codes dominate
- **Lossless** — perfect reconstruction guaranteed
- **Fast** — encoding/decoding is just bit shifts, no complex lookup tables
- **Stream-friendly** — append-only, works naturally for TSDBs

---

## Real World Usage

| System | Technique Used |
|---|---|
| **Facebook Gorilla** | Delta-of-delta + prefix VLE for timestamps |
| **Prometheus** | Gorilla-style delta-of-delta + bit-packing |
| **MongoDB** | Delta + VLE |
| **AWS Redshift** | Delta + VLE |
| **TimescaleDB** | Simple-8b + delta-of-delta + RLE |
| **ClickHouse / Apache IoTDB** | TS_2DIFF (double delta) + varint + bit-packing |
