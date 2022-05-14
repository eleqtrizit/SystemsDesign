# Distributed System Fundamentals

## ACID
In computer science, ACID (atomicity, consistency, isolation, durability) is a set of properties of database transactions intended to guarantee data validity despite errors, power failures, and other mishaps.

### Data Durability & Consistency
The differences and impacts of failure rates of storage solutions and corruption rates in read-write processes.  

Durability guarantees that once a transaction has been committed, it will remain committed even in the case of a system failure (e.g., power outage or crash). This usually means that completed transactions (or their effects) are recorded in non-volatile memory.

## Replication
Reasons why you might want to replicate data:
* To keep data geographically close to your users
* Increase availability
* Increase read throughput

### Leaders and followers
Every write to the database needs to be processed by every replica. The most common solution for this is called _leader-based replication_ (_active/passive_ or _master-slave replication_).
1. One of the replicas is designated the _leader_ (_master_ or _primary_). Writes to the database must send requests to the leader.
2. Other replicas are known as _followers_ (_read replicas_, _slaves_, _secondaries_ or _hot standbys_). The leader sends the data change to all of its followers as part of a _replication log_ or _change stream_.
3. Reads can then query the leader or any of the followers, while writes are only accepted on the leader.

### Synchronous vs asynchronous
The advantage of synchronous replication is that the follower is guaranteed to have an up-to-date copy of the data. **The disadvantage is that it the synchronous follower doesn't respond, the write cannot be processed.**

It's impractical for all followers to be synchronous. If you enable synchronous replication on a database, it usually means that _one_ of the followers is synchronous, and the others are asynchronous.

### Setting up new followers
Setting up a follower can usually be done without downtime. The process looks like:
1. Take a snapshot of the leader's database
2. Copy the snapshot to the follower node
3. Follower requests data changes that have happened since the snapshot was taken
4. Once follower processed the backlog of data changes since snapshot, it has _caught up_.

Process is similar for followers that fell off-line.

### Leader failure: failover
Automatic failover consists:
1. Determining that the leader has failed. If a node does not respond in a period of time it's considered dead.
2. Choosing a new leader. The best candidate for leadership is usually the replica with the most up-to-date changes from the old leader.
3. Reconfiguring the system to use the new leader. The system needs to ensure that the old leader becomes a follower and recognises the new leader.

#### Statement-based replication
The leader logs every _statement_ and sends it to its followers (every `INSERT`, `UPDATE` or `DELETE`).

This type of replication has some problems:
* <span style="color:red">Non-deterministic functions such as `NOW()` or `RAND()` will generate different values on replicas.</span>
* Statements that depend on existing data, like auto-increments, must be executed in the same order in each replica.
* Statements with side effects may result on different results on each replica.

A solution to this is to replace any nondeterministic function with a fixed return value in the leader.

#### Write-ahead log (WAL) shipping
The log is an append-only sequence of bytes containing all writes to the database.  This way of replication is used in PostgresSQL and Oracle. The main disadvantage is that the log describes the data at a very low level (like which bytes were changed in which disk blocks), coupling it to the storage engine.

Usually is not possible to run different versions of the database in leaders and followers.

#### Logical (row-based) log replication
* For an inserted row, the new values of all columns.
* For a deleted row, the information that uniquely identifies that column.
* For an updated row, the information to uniquely identify that row and all the new values of the columns.

Since logical log is decoupled from the storage engine internals, it's easier to make it backwards compatible.

Logical logs are also easier for external applications to parse, useful for data warehouses, custom indexes and caches (_change data capture_).

### Problems with replication lag
In a _read-scaling_ architecture, you can increase the capacity for serving read-only requests simply by adding more followers. However, this only realistically works on asynchronous replication. The more nodes you have, the likelier is that one will be down, so a fully synchronous configuration would be unreliable.

With an asynchronous approach, a follower may fall behind, leading to inconsistencies in the database (_eventual consistency_).

The _replication lag_ could be a fraction of a second or several seconds or even minutes.

### Reading your own writes

_Read-after-write consistency_, also known as _read-your-writes consistency_ is a guarantee that if the user reloads the page, they will always see any updates they submitted themselves.

How to implement it:
* After writing, ensure the user reads from the leader (or synchronous copies)
* The client can remember the timestamp of the most recent write, then the system can ensure that the replica serving any reads for that user reflects updates at least until that timestamp.
* Ensure usage of same datacenter
* You may want to provide _cross-device_ read-after-write consistency for the same user

### Monotonic reads
Ensure the user reads from the same replica, so they do not experience time discrepencies (such as time moving backward due to lag)

### Multi-leader replication
Leader-based replication has one major downside: there is only one leader, and all writes must go through it.

A natural extension is to allow more than one node to accept writes (_multi-leader_, _master-master_ or _active/active_ replication) where each leader simultaneously acts as a follower to the other leaders.

##### Multi-datacenter operation
You can have a leader in _each_ datacenter. Within each datacenter, regular leader-follower replication is used. Between datacenters, each datacenter leader replicates its changes to the leaders in other datacenters.

Compared to a single-leader replication model deployed in multi-datacenters
* **Performance.** With single-leader, every write must go across the internet to wherever the leader is, adding significant latency. In multi-leader every write is processed in the local datacenter and replicated asynchronously to other datacenters. The network delay is hidden from users and perceived performance may be better.
* **Tolerance of datacenter outages.** In single-leader if the datacenter with the leader fails, failover can promote a follower in another datacenter. In multi-leader, each datacenter can continue operating independently from others.
* **Tolerance of network problems.** Single-leader is very sensitive to problems in this inter-datacenter link as writes are made synchronously over this link. Multi-leader with asynchronous replication can tolerate network problems better.

It's common to fall on subtle configuration pitfalls. Autoincrementing keys, triggers and integrity constraints can be problematic. Multi-leader replication is often considered dangerous territory and avoided if possible.

## Consensus
Ensuring all nodes are in agreement, which prevents fault processes from running and ensures consistency and replication of data and processes

### Consistency guarantees
- Do we need data guarantees or is eventual consistency (convergence) OK?
* Stronger guarantees have worse performance

### Linearizability
* Make system _appear_ as if it has only one copy of the data and looks atomic
	* Once new values are read, never return old ones
	* Make data appear serialized -as if they executed in order
	* recency guaranteed on reads and writes
* How different systems are
	* Single-leader repolication is potentially linearizable.
	* Consensus algorithms is linearizable.
	* Multi-leader replication is not linearizable.
	* Leaderless replication is probably not linearizable.
* Multi-leader replication is often a good choice for multi-datacenter replication. On a network interruption between data-centers will force a choice between linearizability and availability.
	* With multi-leader configuraiton, each data center can operate normally with interruptions.
	* **If an application does not require linearizability it can be more tolerant of network problems.**

### Leader election
* All (or voting) nodes must agree on the leader via a lock
* Apache Zookeeper and etcd are often used for distributed locks
* **Voting**
	* Mostly a synchronous operation
	* Votes are held if the slave nodes believe the master is dead
	* If a predefined fixed # of nodes are selected to vote, then you can't just remove these systems from the cluster
	* Network issues, especially across geographic regions, can cause mass voting
	* **Too much voting prevents work from getting done.**

### Ordering and Causality
* Order preserves causality
* Rows must exist before modification
* Detect concurrent writes
* In order to determine casual ordering
	* Database needs to know which copy of the data was read before write
		* use versions
	* with single leader replication, master can just increment a number
	* With multi-leader, you can use something like Lamport timestamps
		* combo of timestamp and node ID where the largest numbers wins.
		* Each node sends out the lamport timetamp on each request
		* Client sends back these timestamps. If another replica gets a future request with a larger timestamp, the server updates.

#### Key points
 -  Linearizability is stronger than causal consistency
	 - Linearizability implies causality
 - Causal consistency is the strongest possible consistency model that does not slow down due to network delays, and remains available in the face of network failures.
	 - The causal order is not a total order
	 - Some elements will be incomparable **WEAKNESS**
	 - Timestamp ordering is not sufficient **WEAKNESS**

### Total Order Broadcast
Total order broadcast is a protocol for exchanging messages between nodes. It requires two safety properties:
* Reliable delivery: No messages are lost a (if a message is delivered to one node, it is delivered to all nodes)
* Totally ordered delivery: Messages are delivered to every node in the same order
* Makes sure every message and write are delivered in the same order
* No guaranteed when message is delivered

**Example:**
* Append a message to the log, tentatively indicating the username you want to claim
 * Read the log, and wait for the message you appended to be delivered back to you
 * Check for any messages claiming the username that you want. If the first message for your desired username is your own message, then you are successful

Consensus services 
* ZooKeeper
* etcd

## Partitioning
Replication, for very large datasets or very high query throughput is not sufficient, we need to break the data up into _partitions_ (_sharding_).

### Partitioning Types
* key-value
	* Good
		* simple
	* Bad
		* suspectible to hot spots
		* no way of knowing what shard the data is on
* key range
	* Good
		* Can be sorted and scanned easily
		* Know where shard is located physically
	* Bad
		* suspectible to hot spots
* hash of key
	* Good
		* avoids hotspots
	* Bad
		* Can't do sorted range queries anymore
		* Must send range queries to all shards

### Secondary Indexes
Each partition maintains its secondary indexes, covering only the documents in that partition (_local index_).

You need to send the query to _all_ partitions, and combine all the results you get back (_scatter/gather_). This is prone to tail latency amplification and is widely used in MongoDB, Riak, Cassandra, Elasticsearch, SolrCloud and VoltDB.

### Global Index
We construct a _global index_ that covers data in all partitions. The global index must also be partitioned so it doesn't become the bottleneck.

The advantage is that it can make reads more efficient: rather than doing scatter/gather over all partitions, a client only needs to make a request to the partition containing the term that it wants. The downside of a global index is that writes are slower and complicated.

## Distributed Transactions
Once consensus is reached, transactions from applications need to be committed across databases with fault checks by each resource involved.

There are situations in which it is important for nodes to agree:
* Leader election: All nodes need to agree on which node is the leader.
* Atomic commit: Get all nodes to agree on the outcome of the transacction, either they all abort or roll back.

### Atomic commit and two-phase commit (2PC)
2PC is also called a _blocking_ atomic commit protocol, as 2Pc can become stuck waiting for the coordinator to recover. Heavy performance penalty.
* If all participants reply "yes", the coordinator sends out a _commit_ request in phase 2, and the commit takes place.
* If any of the participants replies "no", the coordinator sends an _abort_ request to all nodes in phase 2.
* The problem with _locking_ is that database transactions usually take a row-level exclusive lock on any rows they modify, to prevent dirty writes.
* If Coordinator fails, human interaction is required, otherwise orphaned updates and locks may linger in database forever.

### Fault-tolerant consensus
One or more nodes may _propose_ values, and the consensus algorithm _decides_ on those values.

Consensus algorithm must satisfy the following properties:
* Uniform agreement: No two nodes decide differently.
* Integrity: No node decides twice.
* Validity: If a node decides the value _v_, then _v_ was proposed by some node.
* Termination: Every node that does not crash eventually decides some value.

So total order broadcast is equivalent to repeated rounds of consensus:
* Due to agreement property, all nodes decide to deliver the same messages in the same order.
* Due to integrity, messages are not duplicated.
* Due to validity, messages are not corrupted.
* Due to termination, messages are not lost.

## Storage and Retrieval

### Write Log
Most databases use an append-only log.  To avoid the log from growing too big, you can close a log and start a new one.  Then, compact the older logs.

#### Types of Indexes
 Indexes speed up retrieval.
* Hash Indexes
* SSTables
* LSM-Trees
* B-Trees

### Data Warehousing
A _data warehouse_ is a separate database that analysts can query to their heart's content without affecting OLTP operations. It contains read-only copy of the dat in all various OLTP systems in the company. Data is extracted out of OLTP databases (through periodic data dump or a continuous stream of update), transformed into an analysis-friendly schema, cleaned up, and then loaded into the data warehouse (process _Extract-Transform-Load_ or ETL).

#### Column-oriented storage
In a row-oriented storage engine, when you do a query that filters on a specific field, the engine will load all those rows with all their fields into memory, wasting lots of resoruces

_Column-oriented storage_ is simple: don't store all the values from one row together, but store all values from each _column_ together instead. Only load what is needed.  

Column-oriented storage is more CPU effecient and can be compressed for faster loading.


