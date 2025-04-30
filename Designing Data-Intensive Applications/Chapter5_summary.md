## Chapter 5: Replication

Replication in distributed systems refers to the process of keeping copies of the same data across multiple machines. The reasons for replicating data include:

- **Geographical proximity**: Reducing latency by placing data closer to users.
- **Availability**: Allowing the system to continue operating even if some parts fail.
- **Scalability**: Increasing read throughput by allowing more machines to serve read queries.

This chapter focuses on how replication works when the dataset fits within a single machine's storage. For larger datasets, partitioning (sharding) is used, which will be covered in Chapter 6.

### Challenges of Data Replication

Replication is easy when data doesn't change over time. However, the primary challenge lies in handling data changes across replicas. Three popular algorithms for handling these changes are:

1. **Single-Leader Replication** (Active/Passive or Master/Slave)
2. **Multi-Leader Replication**
3. **Leaderless Replication**

Each method has its advantages and trade-offs, especially around consistency, durability, and failure recovery.

### Leader-Based Replication

In leader-based replication, one replica is designated as the **leader** (or master). Clients must send write requests to the leader, which then propagates the data change to its followers. The followers update their local copies based on the leader's change log.

Key features:
- Writes are only accepted by the leader; followers are read-only.
- Databases like **PostgreSQL**, **MySQL**, and **MongoDB** use leader-based replication.
- Non-database systems like **Kafka** and **RabbitMQ** also use this approach.

### Synchronous vs. Asynchronous Replication

**Synchronous replication** ensures that data changes are confirmed by all replicas before the leader notifies the client. This provides stronger consistency guarantees but introduces latency due to waiting for confirmations from all followers.

**Asynchronous replication** is faster because the leader does not wait for follower confirmation. However, data loss is possible if the leader fails before the changes are propagated to followers.

- **Synchronous replication** guarantees consistency but risks unavailability if a follower is slow or down.
- **Asynchronous replication** improves availability and performance but may lead to data loss during failures.

### Failover and Recovery

**Follower failure**: If a follower crashes, it can recover by catching up on missed changes from its log. This allows it to re-sync with the leader without requiring downtime.

**Leader failure**: If the leader fails, one of the followers must be promoted to leader. This is known as **failover**. The new leader may not have all the latest data if the replication was asynchronous, leading to potential data loss. The failover process can be manual or automatic, with automatic failover relying on a **consensus** algorithm to elect a new leader.

### Setting Up New Followers

When adding new followers to a replicated system, a snapshot of the leader’s database is taken. The follower then receives a replication log of all changes that occurred since the snapshot was taken. Once the follower has caught up with the leader, it can continue receiving changes.

### Handling Node Outages

Node outages can be planned (e.g., maintenance) or unplanned (e.g., crashes). Replication systems aim to handle these outages without downtime.

- **Follower failure**: Follower nodes can recover by processing their local log, catching up with the leader.
- **Leader failure**: The system must automatically or manually promote a new leader and reconfigure clients to direct writes to the new leader. This process is prone to problems, such as data loss and conflicting writes, especially in asynchronous systems.

### Replication Logs

Replication often relies on logging mechanisms to keep track of changes. There are different methods for logging and replicating data:

- **Statement-based replication**: The leader logs every SQL statement (e.g., `INSERT`, `UPDATE`, `DELETE`) and sends it to followers. This approach can lead to issues when functions with side effects or nondeterministic operations (like `RAND()`) are executed, as the output may differ across replicas.
  
- **Row-based replication**: Instead of logging statements, the leader logs the actual data changes at the row level. This method is more robust, especially in cases with nondeterministic functions or side effects. Most modern databases, like MySQL, have switched to row-based replication by default.

### Research and Advanced Topics

Researchers continue to improve replication methods to ensure better performance and reliability. **Chain replication**, a variant of synchronous replication, has been successfully implemented in systems like **Microsoft Azure Storage**.

Replication consistency is tightly connected to **consensus algorithms** (discussed in Chapter 9), which are essential for resolving conflicts and ensuring all replicas agree on the correct state.

### Practical Considerations

- **Network issues**: Temporary network problems can delay replication and cause data inconsistencies. Systems must handle these delays gracefully.
- **Consistency vs. Availability**: There is a constant trade-off between maintaining consistency across replicas and ensuring high availability. Asynchronous replication tends to prioritize availability, while synchronous replication prioritizes consistency.

In summary, replication is a critical aspect of distributed systems that involves balancing performance, consistency, and fault tolerance. The various replication strategies (single-leader, multi-leader, leaderless) and configurations (synchronous vs. asynchronous) offer different trade-offs in terms of system availability, data consistency, and failure recovery.

---

# Write-Ahead Log (WAL) Shipping and Replication

### 1. **Write-Ahead Log (WAL) Shipping**
In storage systems, data is typically written to a log first to ensure durability, providing a mechanism to restore data in case of failure. In a **log-structured storage engine**, like SSTables and LSM-Trees, the log is central to the system and compacted in the background. In contrast, a **B-tree** storage engine writes modifications to a **Write-Ahead Log (WAL)** first, before applying them to disk. This log records every modification to disk blocks, ensuring the system's consistency after a crash.

- **Replication with WAL**: WAL can be used to replicate data to another node by transmitting the log across the network to followers. As the follower processes the log, it reconstructs the data structures from the leader. However, the downside of WAL-based replication is that it is tied closely to the storage engine’s internal structure, making it difficult to handle version mismatches between leader and follower nodes. This complicates zero-downtime upgrades and can cause operational issues.

### 2. **Logical (Row-Based) Log Replication**
An alternative to WAL shipping is **logical log replication**, which decouples replication from storage engine specifics. This log describes database writes at the **row level** (e.g., for inserts, deletes, and updates). Each write transaction generates records in the logical log, including information about inserted rows, deleted rows (identified by primary key), and updated rows (including new values).

- **Advantages of Logical Log Replication**:
  - **Decoupling from Storage Engine**: Since the logical log format is separate from the storage engine's structure, it is easier to maintain compatibility between database versions, allowing different versions of software or storage engines on the leader and follower nodes.
  - **External Applications**: Logical logs are also useful for change data capture (CDC), where external systems like data warehouses or custom caches can read the log to replicate changes or build indexes.

### 3. **Trigger-Based Replication**
In some scenarios, more flexibility is required than what traditional replication methods provide. **Trigger-based replication** allows users to write custom application code that is executed automatically when data changes occur in a database. These triggers can log changes into a separate table for external processes to read and replicate.

- **Drawbacks**: Trigger-based replication introduces overheads and is more prone to bugs and complexities compared to the database's built-in replication mechanisms. However, it is a flexible approach for applications that require custom logic or partial data replication.

### 4. **Replication Lag**
Replication introduces the potential for **replication lag**, where the follower nodes may fall behind the leader in reflecting recent writes. This can cause temporary inconsistencies, leading to situations where data is outdated on followers. This phenomenon is known as **eventual consistency**.

- **Leader-Based Replication**: In leader-based replication, writes go through a single leader, but read-only queries can be distributed across followers. To improve read scalability, many followers can be added, but the replication must be asynchronous to avoid making the system unavailable during failures.
- **Replication Lag**: Lag occurs due to network issues, system load, or geographical distance between replicas. The system might return different results for the same query on the leader and follower due to the data not being synchronized. This inconsistency is temporary and resolved as followers eventually catch up with the leader.

### 5. **Handling Replication Lag**
- **Reading Your Own Writes**: A common use case is to ensure that users see the data they just submitted (read-after-write consistency). This can be addressed by reading data from the leader if it was recently modified by the user, while reading other data from a follower.
- **Monotonic Reads**: A monotonic read guarantee ensures that users will not see data "moving backward in time." This is achieved by ensuring that a user reads from the same replica throughout a session or by tracking replication lag and redirecting queries to the leader if necessary.
- **Consistent Prefix Reads**: This guarantee ensures that a sequence of writes is seen in the correct order, preserving causality. Without this, readers may see out-of-order data, such as receiving answers before questions were asked in a conversational scenario.

### 6. **Solutions to Replication Lag Issues**
While **eventual consistency** is often sufficient for certain applications, there are scenarios where stronger guarantees are needed. For example, **read-after-write consistency** ensures users see their updates immediately, while **monotonic reads** and **consistent prefix reads** preserve data integrity by preventing anomalies such as reading stale or out-of-order data.

- **Replication Lag Management**: Applications can manage replication lag by monitoring the lag time and routing requests to the leader if the lag is too significant. In distributed systems, additional complexity arises when replicas are located in multiple data centers or when users access the system from multiple devices, requiring mechanisms like **client timestamps** or **replica assignment** to ensure consistency across devices.

### 7. **Multi-Leader Replication**
In **multi-leader replication** (also called active/active replication), multiple nodes can accept writes, which are then propagated to other nodes. This extends the leader-based model but introduces challenges, such as conflicts between writes happening simultaneously on different leaders.

- **Benefits**: Multi-leader replication provides higher availability and fault tolerance, as any node can serve as a leader for writes.
- **Challenges**: Conflict resolution and consistency issues arise, especially when simultaneous writes conflict. The system needs mechanisms to handle conflicts and ensure data integrity across all nodes.

### Conclusion
Replication strategies, such as **WAL shipping**, **logical log replication**, and **trigger-based replication**, each offer trade-offs in terms of flexibility, compatibility, and consistency. While eventual consistency and asynchronous replication provide scalability, they introduce challenges like replication lag and anomalies. Solutions like **read-after-write consistency**, **monotonic reads**, and **consistent prefix reads** help mitigate these issues, ensuring a more reliable and consistent user experience. In multi-leader systems, conflict resolution mechanisms must be in place to maintain data integrity. As systems scale, understanding and addressing these replication challenges becomes crucial to ensuring smooth operations and a positive user experience.


---
# Use Cases for Multi-Leader Replication

Multi-leader replication allows multiple nodes to accept write operations, replicating changes asynchronously among them. While this model introduces complexity, there are several compelling use cases where it offers tangible benefits.

## Multi-Datacenter Operations

In single-leader setups, all writes must go through one datacenter, introducing latency and reducing availability. Multi-leader replication allows each datacenter to have its own leader. Benefits include:

- **Performance**: Writes are handled locally and asynchronously replicated, hiding network latency.
- **Datacenter Fault Tolerance**: Each datacenter continues to operate independently; replication resumes when connectivity is restored.
- **Network Reliability**: More tolerant of inter-datacenter network issues since writes are not dependent on a remote leader.

However, this configuration requires careful conflict resolution and can be complex to manage.

## Clients with Offline Operation

Applications like calendar apps often need offline support. Each device has a local database acting as a leader, and synchronization happens when connectivity is restored. This is essentially an extreme form of multi-leader replication, where each device is its own "datacenter." CouchDB supports this model natively.

## Collaborative Editing

Tools like Etherpad and Google Docs allow concurrent editing by multiple users. Each user’s client maintains a local state and syncs with the server and other clients asynchronously. This setup:

- Mirrors multi-leader replication
- Requires mechanisms to merge or resolve concurrent updates

Lock-based models resemble single-leader replication, while real-time editing necessitates conflict resolution strategies.

## Handling Write Conflicts

The primary challenge in multi-leader replication is write conflicts. For example, if two users edit the same record simultaneously, both changes may be accepted locally but cause inconsistencies upon replication.

### Synchronous vs Asynchronous Conflict Detection

- **Single-leader**: Conflicts are blocked or retried immediately.
- **Multi-leader**: Both writes succeed and conflict is detected later.

Synchronous conflict detection in multi-leader setups negates the model's benefits.

## Conflict Avoidance

Routing all writes for a particular record through a designated leader can prevent conflicts. This approach works well when users are assigned to specific datacenters. However, reassigning leaders due to failures or user location changes can lead to conflicts.

## Converging to a Consistent State

Multi-leader systems lack a global write order, requiring mechanisms to ensure eventual consistency:

- **Last Write Wins (LWW)**: Chooses the update with the latest timestamp—prone to data loss.
- **Replica Precedence**: Writes from higher-ID replicas override others—also risks data loss.
- **Merge Values**: Combine updates (e.g., "B/C") to preserve both changes.
- **Conflict Recording**: Store conflicting versions and resolve later in application logic.

## Custom Conflict Resolution Logic

Custom resolution can occur:

- **On Write**: Triggered during replication (e.g., Bucardo with Perl snippets).
- **On Read**: Detected conflicts are returned, and application logic resolves them (e.g., CouchDB).

Resolution typically happens at the row or document level, not across entire transactions.

## Automatic Conflict Resolution

Manual conflict resolution is complex and error-prone. Several automatic approaches include:

- **CRDTs (Conflict-Free Replicated Data Types)**: Structures like sets, counters, and maps that auto-resolve conflicts.
- **Mergeable Persistent Data Structures**: Track history and use three-way merges (like Git).
- **Operational Transformation (OT)**: Used in collaborative editors to handle concurrent text edits.

These methods are promising for simplifying multi-leader replication.

## What is a Conflict?

Conflicts can be obvious (e.g., two edits to the same field) or subtle (e.g., two bookings for the same room at the same time). The application must determine what constitutes a conflict and how to handle it appropriately.

----

## Multi-Leader Replication Topologies

Multi-leader replication allows multiple nodes to accept writes, requiring careful coordination to replicate data consistently. The topology defines how writes propagate between nodes. With two leaders, they simply exchange updates. With more than two, multiple configurations arise:

- **All-to-all**: Every leader communicates with all others.
- **Circular**: Each node forwards its writes to the next node in a loop.
- **Star**: One central node distributes writes to others; extendable to tree structures.

In circular and star topologies, writes traverse multiple nodes, requiring forwarding of data changes. Each write is tagged with node identifiers it has visited to avoid replication loops. These topologies suffer from single points of failure, where one node's failure can break the replication chain. All-to-all topologies offer better fault tolerance but may introduce out-of-order message delivery due to network variability.

### Causality and Conflict Detection

Write order matters, especially when updates depend on previous inserts. Simply timestamping writes is insufficient due to clock skew. Instead, **version vectors** can help maintain causality. Unfortunately, many systems lack robust conflict resolution. For instance, PostgreSQL BDR lacks causal ordering, and Tungsten Replicator doesn’t detect conflicts.

Thorough documentation review and testing are essential when using multi-leader systems.

## Leaderless Replication

Leaderless replication discards the concept of a central authority. Any node can accept writes. This approach was revitalized by Amazon’s Dynamo and is employed in Riak, Cassandra, and Voldemort.

Writes are sent to multiple replicas, either directly by the client or via a coordinator that doesn't impose order. If some nodes are down, writes can still proceed as long as a quorum of acknowledgments is achieved.

### Handling Node Failures

If a node is down during a write, available nodes still accept the write. When the offline node returns, it may serve stale data. To mitigate this:

- **Read repair**: Upon detecting stale reads, clients update outdated replicas.
- **Anti-entropy**: A background process reconciles data between nodes, ensuring eventual consistency.

Not all systems implement both. For instance, Voldemort lacks anti-entropy, risking data loss for rarely read items.

## Quorums in Leaderless Replication

Writes and reads use configurable quorum settings:

- **n**: Total replicas
- **w**: Required write acknowledgments
- **r**: Required read responses

The condition `w + r > n` ensures at least one node overlaps in read and write operations, increasing the likelihood of reading up-to-date values. Typical configurations:

- **n = 3, w = 2, r = 2**: Tolerates one failure.
- **n = 5, w = 3, r = 3**: Tolerates two failures.

Depending on the workload, you may choose:

- **High-read throughput**: `w = n, r = 1`
- **Balanced consistency**: `w = r = (n + 1)/2`

Note that the dataset may be partitioned, with each key stored on only `n` nodes.

## Limitations of Quorum Consistency

Even with `w + r > n`, stale reads can occur due to:

- **Sloppy quorums**: Writes may be redirected, breaking the quorum overlap.
- **Concurrent writes**: Without clear ordering, data conflicts may arise.
- **Concurrent reads and writes**: In-flight writes may lead to indeterminate reads.
- **Partial write failures**: Some nodes may persist data even if the quorum is not reached.
- **Replica restoration issues**: Restoring from stale nodes can reduce the count of up-to-date replicas below `w`.

In practice, quorum settings reduce, but don’t eliminate, the risk of anomalies. Applications needing strong guarantees must implement higher-level mechanisms like consensus or transactions.

## Monitoring Staleness

Monitoring staleness is crucial for reliability:

- **Leader-based systems**: Use replication log positions to measure lag.
- **Leaderless systems**: Harder to measure due to lack of ordered logs. Without anti-entropy, infrequently accessed values may remain stale indefinitely.

Research continues into quantifying staleness and predicting stale read likelihood based on quorum settings. Awareness of replication lag helps diagnose underlying issues such as network latency or overloaded nodes.

---

# Detecting Concurrent Writes in Dynamo-style Databases

Dynamo-style databases allow multiple clients to concurrently write to the same key, leading to inevitable write conflicts. These conflicts can occur even when using strict quorum protocols and may arise during operations like read repair or hinted handoff. Network delays and partial failures further complicate this, as events may arrive in different orders across different nodes.

For example, consider a scenario where clients A and B write concurrently to key X across a three-node datastore:
- **Node 1** receives A's write but misses B's due to a transient outage.
- **Node 2** receives A's write, then B's.
- **Node 3** receives B's write, then A's.

If nodes naively overwrite values with each new write, inconsistencies arise: some believe the final value is A, others B. To ensure **eventual consistency**, replicas must **converge to the same value**.

## Conflict Resolution Techniques

Most databases do not resolve conflicts automatically; instead, the application developer must manage conflict resolution. Some common approaches include:

### Last Write Wins (LWW)
LWW resolves conflicts by keeping the "most recent" value based on an attached timestamp, discarding others. While this guarantees convergence, it may:
- **Lose data**, since concurrent writes are silently dropped.
- Discard even non-concurrent writes due to clock inaccuracies.

LWW is used in systems like Cassandra and optionally in Riak. It is best suited for immutable keys or scenarios like caching, where data loss is tolerable. The safest pattern with LWW is to use unique keys for each write (e.g., UUIDs).

## Understanding Concurrency: The "Happens-Before" Relationship

To determine if two operations are concurrent:
- **A happens before B** if B knows or depends on A.
- If neither knows about the other, they are **concurrent**.

Concurrency does not depend on exact timestamps but on causal relationships. Network delays can cause operations to be concurrent even if time allows communication.

### Capturing Happens-Before with Versioning

An example of a shopping cart demonstrates this:
- Each write includes the **version number** of the data it read.
- The server assigns a new version and retains concurrent versions (called **siblings**).
- Clients merge these siblings during the next write.

#### Algorithm Summary:
- Server maintains version numbers per key.
- Clients read before writing and include version info.
- Writes overwrite all versions up to the provided version but retain higher (concurrent) ones.

This ensures no data is silently lost, but it shifts the burden of **merging siblings** to the client.

### Merging Siblings

Conflict resolution during merging depends on application logic:
- **Union merge** for additive operations like shopping carts.
- For deletions, use **tombstones** (deletion markers) to prevent resurrection of removed items.
- Systems like **Riak** use **CRDTs** to automate conflict resolution and merge logic.

## Handling Writes in Multi-Replica Environments

Single version numbers are insufficient in multi-replica settings. Instead, **version vectors** track version numbers per replica.

- Each replica increments its own version and tracks versions from other replicas.
- This collection of versions is called a **version vector**.
- **Dotted version vectors** (used in Riak 2.0) provide fine-grained tracking.

Clients receive version vectors during reads and must send them back during writes. These vectors help determine if writes are overwrites or concurrent. The structure ensures:
- Safe reads from one replica.
- Safe writes to another.
- Preservation of data even in the face of concurrent updates.

## Vector Clocks vs. Version Vectors

While sometimes used interchangeably, **vector clocks** and **version vectors** are subtly different. For practical purposes, **version vectors** are the preferred structure when comparing replica states.

---

In conclusion, handling concurrent writes in Dynamo-style systems involves:
- Detecting concurrency through versioning.
- Retaining conflicting versions.
- Requiring intelligent merge strategies.
- Using tools like version vectors and CRDTs to ensure durability and correctness.

These mechanisms are essential for building reliable distributed systems that support high availability and eventual consistency.
