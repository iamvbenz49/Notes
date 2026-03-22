# JunoDB — PayPal's Distributed Key-Value Store

---

## What is JunoDB?

JunoDB is a **distributed key-value store** developed and open-sourced by PayPal. It is designed to be highly efficient for both **storing and caching data**, reducing the load on relational databases.

- Written in **Golang** to leverage high concurrency via goroutines
- Supports both **in-memory (temporary)** and **on-disk (persistent)** storage
- Configurable **TTL (Time-to-Live)** ranging from seconds to days

---

## JunoDB vs Redis

| | Redis | JunoDB |
|---|---|---|
| Threading model | Single-threaded | Multi-threaded (goroutines) |
| CPU utilization | One core only | All available cores |
| Best for | Simple caching | CPU-bound + high concurrency workloads |

Redis is primarily single-threaded, meaning it only utilizes one CPU core effectively. JunoDB was designed from the ground up to be multi-threaded, making it better suited for PayPal's scale.

---

## Performance & Reliability

- **Availability:** Six nines — **99.9999%** uptime for PayPal
- **Scale:** Handles **350 billion requests daily**

---

## Use Cases

### 1. Caching User Auth Tokens

Authentication tokens are cached to validate user sessions quickly without hitting the primary authentication database. Supports configurable TTL so tokens expire automatically.

### 2. Storing Account Details

Account information that does not update frequently is cached to reduce read load on relational databases. Faster retrieval without repeatedly querying Oracle.

### 3. API Response Caching

Responses from downstream microservices are cached to prevent overloading other services. If a service goes down briefly, the cached response can still be served.

### 4. User Preferences

Near-static data such as notification settings and display preferences is stored in JunoDB for fast retrieval without database roundtrips.

### 5. Idempotency Key Storage

Ensures financial transactions are **processed exactly once**, even if a request is retried multiple times. JunoDB stores idempotency keys so duplicate payment requests or notifications are detected and rejected — no double charges or duplicate transfers.

---

## Latency Bridging

### The Problem

PayPal runs Oracle databases in an **active-active setup** across geographically dispersed data centers. Traditional database replication between these data centers is too slow — creating significant **replication lag**.

This means a write made in one data center may not be immediately visible to a read in another, breaking consistency in a global active-active setup.

### The JunoDB Solution

JunoDB is deployed **alongside Oracle** to bridge this gap:

- Offers **near-instant inter-cluster replication** across data centers
- A write in one cluster is immediately available for reads in another
- Bridges the latency gap without sacrificing write throughput
- Enables true **active-active database setups** at PayPal's scale

```
Data Center A          Data Center B
┌─────────────┐        ┌─────────────┐
│   Oracle    │──slow──│   Oracle    │
│   JunoDB   │◄─fast──►│   JunoDB   │
└─────────────┘        └─────────────┘
     write                  read
  (near-instant replication via JunoDB)
```
