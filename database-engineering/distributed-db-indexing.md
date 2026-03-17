# Distributed Database Indexing — Video Notes

---

## What is Indexing?

Indexing is a technique used to make database lookups faster, typically by creating an index on secondary attributes. In a **distributed database** that is sharded or partitioned, indexing becomes complex because data is spread across multiple nodes.

---

## Partition Key

A partition key is the fundamental attribute used to **shard (partition) data** and distribute it across multiple nodes.

- **Data Distribution** — The key is passed through a hash function to determine which node stores a piece of data (e.g., using `author_id` as the key).
- **Efficient Querying** — Queries that include the partition key are fast because the database proxy knows exactly which shard holds the data and routes directly to it.
- **Fan-out Risk** — Querying on an attribute that is *not* the partition key (e.g., `category`) forces the database to send the request to **all nodes**, which is slow and expensive.

### Risks of Querying Without the Partition Key

- **Slow Shard Impact** — Even if shards are queried in parallel, the entire response is delayed by the slowest shard.
- **Dead Shard Risk** — If a shard is down, the system may timeout or return incomplete results.
- **High Network Load** — Querying all shards causes excessive data transfer across the network, most of which gets filtered out afterward.

---

## Global Secondary Index (GSI)

A GSI is a **separate index** used to optimize queries on attributes that are not the primary partition key (e.g., querying by `category` instead of `author_id`).

### How It Works

1. The GSI is **partitioned by the secondary attribute** (e.g., `category`).
2. When a query is fired, the proxy hashes the category to find the correct **GSI shard**.
3. The GSI shard is scanned to retrieve the **primary keys** (e.g., blog IDs) for that category.
4. The proxy then contacts the relevant **primary data shards** to fetch the full objects.

This avoids fan-out — only the relevant shards are contacted instead of all of them.

### Data Storage Options

| Option | What's Stored in GSI | Lookup Steps | Trade-off |
|---|---|---|---|
| **Primary Key Reference** | Category + Blog ID only | 2 lookups (GSI shard → data shard) | Smaller index, slower reads |
| **Full Attribute Storage** | Entire blog object | 1 lookup (GSI shard only) | Faster reads, larger index size |

### Logical Separation

While GSI shards may be co-located with existing data nodes, there is a **logical separation** between where the primary data lives and where the index lives.

### Keeping GSIs in Sync

- Most databases offer **strong consistency** for indexes — any update to a document must be **synchronously applied** to the GSI as well.
- **High Cost of Writes** — Every write must update the main data shard *and* all corresponding GSI shards.
- **Limited Number of GSIs** — Due to write performance impact, databases typically restrict GSIs to **5–7 per table**.

---

## Local Secondary Index (LSI)

An LSI is an index **localized to a single shard**, used when queries always include the partition key.

### How It Works

- Used when you need to query a secondary attribute (e.g., `category`) but only within the scope of a specific partition key (e.g., `user_id`).
- Since the partition key is always included, the proxy knows exactly which node to contact — **no fan-out needed**.
- Index data and main data reside on the **same shard**, making strong consistency easier to maintain.

### Limitations

- **Limited Scope** — Cannot efficiently query across the entire database for a secondary attribute without the partition key.
- **Mandatory Partition Key** — Every query using an LSI *must* include the partition key. You cannot query by category alone.
- **Not a Replacement for GSI** — If your query pattern requires searching a secondary attribute across all nodes, you need a GSI instead.

---

## GSI vs LSI — Quick Comparison

| | GSI | LSI |
|---|---|---|
| Scope | Entire cluster | Single shard |
| Requires partition key | No | Yes |
| Consistency | Strong (costly) | Strong (easier) |
| Avoids fan-out | Yes | Yes |
| Write cost | High | Lower |
| Cross-shard queries | Yes | No |
