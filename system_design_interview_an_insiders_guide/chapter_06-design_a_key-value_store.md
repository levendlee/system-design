No-relational database. Keys are unique. Values are opaque.

# Understand the problem and establish design scope
Tradeoffs: Read/Write/Memory Usage; Consistency/Availability.
Target design:
- Size of k-v pair is small. Less than 10KB.
- Ability to store big data.
- High availability: The system responses quickly, even during failures.
- High scalability: The system can be scaled to support large data set.
- Automatic scaling: The addition/deletion of servers should be automatic based on traffic.
- Tunable consistency.
- Low latency.

# Single server
Use hash table. However, space constraints.
- Data compression.
- Store frequently used data in memory and the rest on disk.

# Distributed
Distributed hash table across many servers.

## CAP theorem
Consistency, Availability, Partition tolerance. CAP theorem states impossible ti simultaneously provide more than two of these three guarantees.

### Consistency
All clients see the same data at the same time no matter which node they connect to.
### Availability
Availability means any client which requests data gets a response even if some of the nodes are down.
### Partition Tolerance
A partition indicates a communication break between two nodes. Partition tolerance means the system continues to operate despite network partitions.

Categorized based on two CAP characteristics they support:
- CP systems: Supports consistency and partition tolerance. Sacrifice availability. 
- AP systems: Supports availability and partition tolerance. Sacrifice consistency.
- CA systems: Support consistency and availability. Sacrifice partition tolerance. Not doable in practice as network failures always happen.

## Ideal solution

3 replicas. Network partition never occurs. Data written to n1 is automatically replicated to n2 and n3. Consistency and availability are achieved.

## Real-world distributed system

Network partition cannot be avoid.
- CP: Blocks all write operations to n1 and n2 to avoid data inconsistency, making the unavailable. (e.g Banking)
- AP: Keeps accepting reads, even though it might return stale data.

## System components

### Data partition
Challenges:
- Distribute data across multiple servers evenly.
- Minimize data movement when nodes are added or removed.
Use consistent hashing:
- Automatic scaling: Servers could be added and removed automatically depending on the load.
- Heterogeneity: The number of virtual nodes for a server is proportional to the server capacity. Server with higher capacity are assigned with more virtual nodes.

### Data replication
To replicate across N servers, walk clockwise from the position in hash ring and fine first N unique servers to replicate. Better to choose N unique data centers.

### Consistency
Quorum consensus to guarantee consistency for both read and write operations.
Read/Write operations are always sent to all replicas.
- *N*: The number of replicas.
- *W*: A write quorum of size W. For a write operation to be successful, write operation must be acknowledged from W replicas.
- *R*: A read quorum of size R. For a read operation to be successful, read operation must wait for responses for at least R replicas.

- If W=1 or R&gt;1, then better latency.
- If W&gt;1 or R&gt;1, then better consistency.
- If W + R &gt; N, strong consistency is guaranteed because there must be at least one overlapping node that has the latest data to ensure consistency.

Typical setups:
- R=1, W=N. System optimized for fast read.
- W=1, R=N. System optimized for fast write.
- W+R&gt;N, strong consistency guaranteed. Usually N=3, W=R=2.
- W+R&lt;=N, strong consistency not guaranteed.

### Consistency models
- *Strong consistency*: Any read operation returns a value corresponding to the result of the most updated write data item. Never out-of-date data in client.
- *Weak consistency*: Subsequent read options may not see the most updated value.
- *Eventual consistency*: Special form of weak consistency. Given enough time, all updated are propagated and all replicas are consistent.

Strong consistency usually achieved by forcing a replica not to accept new read/writes until every replica has agreed on current write. Not ideal for highly available systems. 
Dynamo and Cassandra adopt eventual consistency, recommended. Form concurrent writes, allows inconsistent values to enter the system and force the client to read values to reconcile.

### Inconsistency resolution: Versioning

A version close is a [server, version] pair associated with a data item. If data item D is written to server Si, the system must perform one of the following tasks:
- Increment vi if [Si, Vi] exists.
- Otherwise, create a new entry [Si, 1].

**Client detect conflicts and resolve them by adding a new write**. Conflicts happens when **X isn't an ancestor but a sibling of Y**, AKA, neither X's all versions on all servers are strictly larger or equal than Y, or X's all versions on all servers are strictly smaller or equal than Y.

Cons
- Version clocks add complexity to client side as it needs to resolve conflicts.
- Version clocks in server could grow rapidly. Need a threshold. If exceeds, the oldest pairs are removed, leading to inefficiencies in reconciliation because the descendant relationship cannot be determined correctly.

## Handling failures
### Failure detection
Requires at least two independent sources of information to mark a server down. All-to-all multicasting is straightforward solution but inefficient when many servers are in the system. Better de-centeralized.

**Gossip protocol**
- Each node maintains a node membership list contains IDs and heartbeat counters.
- Each node periodically increments its hearbeat counter.
- Each node periodically sends heartbeats to a set of random nodes, in turn propagate to another set of nodes.
- Once nodes receive heartbeats, membership list is updated to the latest info.
- If heartbeat not increased for more than pre-defined periods, the member is considered as offline.

### Handling temporary failures
- Strict quorum: read and write are blocked.
- Sloppy quorum: chooses first W/R healthy servers for write/read. Ignore offline. Hinted handoff: another server will process requests temporarily and push back changes to achieve data consistency.

### Handling permanent failures
Implements an anti-entropy protocol to keep replicas in sync.

**Hash tree** (Merkle tree)
- A binary tree which each nodes contains the hash of the subtree, the leaf nodes contains the hash of buckets.
- Synchronization is proportional to the differences between the two replicas, not the amount of data they contain. Bucket size is quite big in real-word.

### Handling data center outage
Replica accorss multiple data centers.

## System architecture diagram
- Main features
  - Clients communicate through simple API.
  - A coordinator acts as a proxy between client and k-v store.
  - Nodes are distributed on ring using consistent hashing.
  - System decentralized so adding/moving is automatic.
  - Data is replicated at multiple nodes.
  - There is no single point of failure as all node as the same set of responsibilities.

- Write path
  - Client  Memory: Cache -&gt; Disk (Commit Log + SSTables (Sorted-String table))/

- Read path
  - Client -&gt; Memory: Cache
  - Client -&gt; Memory: Cache -&gt; Bloom filter -&gt; SSTables -&gt; Client

# Summary
![Summary](https://levendlee.files.wordpress.com/2023/12/screenshot-2023-12-28-at-21.36.38.png)
