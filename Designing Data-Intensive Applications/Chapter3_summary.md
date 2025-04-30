# Chapter 3: Storage and Retrieval – Summary (750 Words)

## Overview

In data-intensive applications, databases play a fundamental role in **storing and retrieving data** efficiently. Understanding how databases manage this process is crucial for choosing the right database, tuning performance, and handling scale. This chapter focuses on the **internal mechanisms** of storage engines and indexing, with a particular emphasis on how different design choices affect performance and capabilities.

---

## Data Storage Using Logs

### Append-Only Logs
Many databases use an **append-only log** for write operations. When a value is written, it is appended to the end of a file along with its key. This approach is simple, ensures durability, and is efficient for sequential writes.

### Problem: Inefficient Reads
However, reading the value associated with a key becomes inefficient, especially as the log grows. A full scan may be required, which has **O(n)** time complexity.

### Example: Bash Key-Value Store
```bash
db_set () {
  echo "$1,$2" >> database
}

db_get () {
  grep "^$1," database | sed -e "s/^$1,//" | tail -n 1
}
```
This simplistic store appends every key-value pair to a log file and retrieves the latest value by scanning. It demonstrates the need for **indexing** in real-world applications.

---

## Indexing Basics

### What is an Index?
An **index** is a data structure that enables efficient data retrieval by avoiding full scans. It maps keys to locations (offsets) in the data file.

### Trade-offs
- **Improves read performance**.
- **Adds write overhead**, as the index must be updated.
- Must be designed based on the application's query patterns.

---

## Hash Indexes

### How It Works
- Uses a **hash map** (often in memory) to map keys to byte offsets in a log file.
- Lookups jump directly to the data location.

### Advantages
- Fast and simple for **exact match** queries.
- Efficient for write-heavy workloads.

### Limitations
- Cannot support **range queries**.
- The entire index must **fit in memory**.
- May degrade if memory is limited or keys are non-uniform.

---

## Sorted String Tables (SSTables)

### Concept
- Maintain log entries in **sorted order by key**.
- Enable **binary search** and efficient **range scans**.
- Support **compaction** and **merging** for storage efficiency.

### Compaction
- Periodically merge segments, discard overwritten or deleted keys.
- Improves read performance and reclaims disk space.

### In-Memory Table (Memtable)
- Writes go to an in-memory structure (e.g., red-black tree).
- When full, the memtable is flushed to disk as a new SSTable.

---

## Log-Structured Merge Trees (LSM-Trees)

### Architecture
- Sequence of SSTables and a memtable.
- New writes go to memtable → flushed to disk → compacted.

### Benefits
- Write-optimized: **sequential disk writes**, batching.
- Good for **write-heavy workloads**.

### Drawbacks
- Reads may involve checking multiple SSTables.
- Compaction overhead must be managed.

---

## B-Trees

### Structure
- Tree-like data structure with sorted keys.
- Nodes contain multiple keys and pointers to child nodes.
- Designed to minimize disk I/O by maximizing data per node.

### Operations
- **Search**: Traverse from root to leaf.
- **Insert**: Place in appropriate leaf, split if full.

### Advantages
- Balanced tree ensures **logarithmic search time**.
- Efficient for **both reads and writes**.
- Supports **range queries** natively.

### Disadvantages
- Write amplification due to random I/O.
- More complex to implement and maintain.

---

## Comparison: LSM-Trees vs. B-Trees

| Feature               | LSM-Trees              | B-Trees               |
|-----------------------|-------------------------|------------------------|
| Write Performance     | High (sequential writes) | Moderate (random I/O) |
| Read Performance      | Moderate (multi-table lookups) | High (single tree lookup) |
| Range Queries         | Possible                | Efficient              |
| Write Amplification   | Can be high (compaction) | Moderate               |
| Implementation        | Simple to moderate       | Complex                |

---

## Optimizations and Variants

### Bloom Filters
- Probabilistic data structure to check **whether a key might exist**.
- Reduces unnecessary disk reads in SSTables.

### Prefix Compression
- Store only differences between keys.
- Saves disk space, especially with long or repetitive keys.

### Copy-on-Write B-Trees
- Avoid in-place updates by writing changes to new pages.
- Improves crash recovery and supports snapshots.

---

