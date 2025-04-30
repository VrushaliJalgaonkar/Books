# Chapter 3: Storage and Retrieval

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
- **Improves read performance**
- **Adds write overhead**, as the index must be updated
- Must be designed based on the application's query patterns

---

## Hash Indexes

### How It Works
- Uses a **hash map** (often in memory) to map keys to byte offsets in a log file
- Lookups jump directly to the data location

### Advantages
- Fast and simple for **exact match** queries
- Efficient for write-heavy workloads

### Limitations
- Cannot support **range queries**
- The entire index must **fit in memory**
- May degrade if memory is limited or keys are non-uniform

---

## Sorted String Tables (SSTables)

### Concept
- Maintain log entries in **sorted order by key**
- Enable **binary search** and efficient **range scans**
- Support **compaction** and **merging** for storage efficiency

### Compaction
- Periodically merge segments, discard overwritten or deleted keys
- Improves read performance and reclaims disk space

### In-Memory Table (Memtable)
- Writes go to an in-memory structure (e.g., red-black tree)
- When full, the memtable is flushed to disk as a new SSTable

---

## Log-Structured Merge Trees (LSM-Trees)

### Architecture
- Sequence of SSTables and a memtable
- New writes go to memtable → flushed to disk → compacted

### Benefits
- Write-optimized: **sequential disk writes**, batching
- Good for **write-heavy workloads**

### Drawbacks
- Reads may involve checking multiple SSTables
- Compaction overhead must be managed

---

## B-Trees

### Structure
- Tree-like data structure with sorted keys
- Nodes contain multiple keys and pointers to child nodes
- Designed to minimize disk I/O by maximizing data per node

### Operations
- **Search**: Traverse from root to leaf
- **Insert**: Place in appropriate leaf, split if full

### Advantages
- Balanced tree ensures **logarithmic search time**
- Efficient for **both reads and writes**
- Supports **range queries** natively

### Disadvantages
- Write amplification due to random I/O
- More complex to implement and maintain

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
- Probabilistic data structure to check **whether a key might exist**
- Reduces unnecessary disk reads in SSTables

### Prefix Compression
- Store only differences between keys
- Saves disk space, especially with long or repetitive keys

### Copy-on-Write B-Trees
- Avoid in-place updates by writing changes to new pages
- Improves crash recovery and supports snapshots

---

## Making an LSM-tree out of SSTables and Comparing with B-Trees

### Overview of LSM-Trees
Log-Structured Merge-Trees (LSM-Trees) are used in modern key-value storage engines like LevelDB, RocksDB, Cassandra, and HBase. They use a combination of in-memory and on-disk data structures:

- **Memtable**: An in-memory structure for fast writes
- **SSTables**: Immutable sorted files written to disk from memtables
- **Compaction**: Merging of SSTables to manage space and optimize reads

LSM-trees store data in sorted order and merge sorted files in the background, allowing for high write throughput and efficient range queries.

### Lucene's Similar Approach
Lucene, used in Elasticsearch and Solr, applies a similar technique for managing its inverted index:
- Stores terms and their corresponding posting lists (document IDs)
- Uses SSTable-like files that are periodically merged

### Performance Optimizations

#### Improving Non-Existent Key Lookups
- **Bloom Filters** help reduce disk reads by quickly identifying missing keys

#### Compaction Strategies
- **Size-Tiered Compaction**: Merges smaller SSTables into larger ones
- **Leveled Compaction**: Organizes SSTables into levels with non-overlapping key ranges

Despite complexities, LSM-trees work well at scale, especially for write-heavy workloads.

---

## B-Trees: A Different Indexing Philosophy

B-trees remain the dominant indexing structure in relational databases. They:
- Organize data into **fixed-size pages** (e.g., 4 KB)
- Use **page references** to build a tree structure with a root
- Allow fast lookups and range scans with high branching factors

### How B-Trees Work
- Keys guide lookups down the tree from internal nodes to leaf pages
- Insertion may trigger **page splits**
- Depth remains logarithmic (typically 3–4 levels deep for millions of keys)

### Ensuring Reliability
- **Write-Ahead Log (WAL)** ensures crash recovery
- Updating in-place requires careful concurrency control (latches/locks)

---

## B-Tree Optimizations

- **Copy-on-Write** (used in LMDB)
- **Key Abbreviation** for saving space
- **Sequential Page Layout** for better scan performance
- **Sibling Pointers** for ordered scans
- **Fractal Trees**: Hybrid of B-trees and LSM-trees

---

## Comparing B-Trees and LSM-Trees

| Feature                 | B-Trees                       | LSM-Trees                            |
|------------------------|-------------------------------|--------------------------------------|
| Write Throughput       | Moderate, write-amplified     | High, sequential writes              |
| Read Performance       | Fast for point queries        | Slower due to multiple SSTables      |
| Space Usage            | Fragmentation possible        | Better compression, less overhead    |
| Crash Recovery         | Needs WAL                     | Appends and atomic segment swaps     |
| Concurrency            | Needs latches                 | Background merging simplifies access |
| Compaction Overhead    | None                          | Can interfere with reads/writes      |
| Storage Layout         | Page-based                    | SSTable segments                     |
| Transaction Support    | Stronger, lockable ranges     | Harder due to key duplication        |

---

## Other Indexing Structures - Summary

### Primary vs. Secondary Indexes
- **Primary index**: Uniquely identifies rows/documents/vertices
- **Secondary index**: Allows access via non-primary fields

#### Implementations
- B-trees
- Log-structured indexes
- Handle duplicates by storing row lists or appending row IDs

### Storing Values in Indexes
- **Heap file**: Index points to data stored separately
- **Clustered index**: Stores full row within index (e.g., InnoDB)
- **Covering index**: Includes non-key columns
- Trade-off: Better read performance vs. higher write/storage cost

### Multi-Column Indexes
- **Concatenated**: Combine fields (e.g., `(last_name, first_name)`)
- **Multi-dimensional**: For geospatial/complex data (e.g., R-trees, HyperDex)

### Full-Text Search and Fuzzy Indexes
- Full-text search supports fuzzy matching, synonyms, etc.
- Uses **Levenshtein automata** in Lucene
- ML-based and edit-distance search possible

### In-Memory Indexing
- RAM removes many disk constraints
- Examples: Redis, VoltDB, Memcached, TimesTen
- Techniques: logging, replication, anti-caching, NVM

---

## OLTP vs. OLAP

### OLTP (Online Transaction Processing)
- Access: Small number of records, low-latency
- Users: End users
- Data: Current state
- Size: GB–TB

### OLAP (Online Analytical Processing)
- Access: Large scans, aggregations
- Users: Analysts
- Data: Historical
- Size: TB–PB

### Example Queries
- Revenue by store
- Sales during promotion
- Product bundling patterns

### Data Warehousing
- Offloads analytics from OLTP
- **ETL**: Extract, Transform, Load
- Uses star/snowflake schemas
- Column-oriented storage for compression and performance

### Columnar Techniques
- **Compression**: Bitmap, RLE
- **Vectorized Execution**: CPU-efficient processing
- **Sorted Columns**: Improves I/O and compression

### Writing to Column Stores
- Buffered writes
- LSM-like merges
- Query planner handles hybrid reads

---

## Summary

OLTP and data warehouses differ in:
- Workloads: transactional vs. analytical
- Storage: row-oriented vs. columnar
- Techniques: indexes, compression, buffering

These innovations allow data systems to scale efficiently while supporting both real-time operations and complex analysis.

