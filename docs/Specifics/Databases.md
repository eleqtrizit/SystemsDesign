# Databases

## What is the difference between MyISAM and InnoDB?

### MyISAM
-  Supports table level locking
-  Is Faster than InnoDB
-  Does not support foreign keys
-  Stores table, data and index in different files
-  Does not support transactions (no commit with rollback)
-  Useful for more selects with fewer updates  

### InnoDB
-  Support row level locking
-  Is slower than MyISAM
-  Supports foreign keys
-  Stores table and index in table space
-  Supports transactions
	- A transaction is a unit of work that you want to treat as "a whole." It has to either happen in full or not at all.
	- e.g move money from one account to another.  Can't let this be half finished.

## What are some things to check on a slow database?
MySQL is fairly popular, so let’s look at some basic MySQL debugging. First off, check the OS to make sure the system is running fine, specially check CPU, memory, SWAP space and disk I/O. 

Assuming those are all ok, then log into MySQL and check the running queries, you can do so by running the command `show full processlist`. This will give you a list of queries running on the server. If you see a query that has been running for an excessively long time, you should investigate that query.

Check
	* `Explain` query
	* check slow query log file
	* Check query cache


## ACID
**ACID** is a set of properties of relational database [transactions](https://en.wikipedia.org/wiki/Database_transaction).
-   **Atomicity** - Each transaction is all or nothing
-   **Consistency** - Any transaction will bring the database from one valid state to another
-   **Isolation** - Executing transactions concurrently has the same results as if the transactions were executed serially
-   **Durability** - Once a transaction has been committed, it will remain so


## Replication
### Tuning
There are many techniques to scale a relational database: **master-slave replication**, **master-master replication**, **federation**, **sharding**, **denormalization**, and **SQL tuning**.

### Master-slave replication

The master serves reads and writes, replicating writes to one or more slaves, which serve only reads. Slaves can also replicate to additional slaves in a tree-like fashion. If the master goes offline, the system can continue to operate in read-only mode until a slave is promoted to a master or a new master is provisioned.
![[Pasted image 20220417103311.png]]


### Master-master replication
Both masters serve reads and writes and coordinate with each other on writes. If either master goes down, the system can continue to operate with both reads and writes.

##### Disadvantage(s): master-master replication

-   You'll need a load balancer or you'll need to make changes to your application logic to determine where to write.
-   Most master-master systems are either loosely consistent (violating ACID) or have increased write latency due to synchronization.
-   Conflict resolution comes more into play as more write nodes are added and as latency increases.
![[Pasted image 20220417103446.png]]

### Disadvantage for all replication
-   There is a potential for loss of data if the master fails before any newly written data can be replicated to other nodes.
-   Writes are replayed to the read replicas. If there are a lot of writes, the read replicas can get bogged down with replaying writes and can't do as many reads.
-   The more read slaves, the more you have to replicate, which leads to greater replication lag.
-   On some systems, writing to the master can spawn multiple threads to write in parallel, whereas read replicas only support writing sequentially with a single thread.

## Federation
Federation (or functional partitioning) splits up databases by function. For example, instead of a single, monolithic database, you could have three databases: **forums**, **users**, and **products**, resulting in less read and write traffic to each database and therefore less replication lag. Smaller databases result in more data that can fit in memory, which in turn results in more cache hits due to improved cache locality. With no single central master serializing writes you can write in parallel, increasing throughput.

##### Disadvantage(s): federation
-   Federation is not effective if your schema requires huge functions or tables.
-   You'll need to update your application logic to determine which database to read and write.
-   Joining data from two databases is more complex with a [server link](http://stackoverflow.com/questions/5145637/querying-data-by-joining-two-tables-in-two-database-on-different-servers).

## Sharding
Sharding distributes data across different databases such that each database can only manage a subset of the data.  If one shard goes down, the other shards are still operational, although you'll want to add some form of replication to avoid data loss.

##### Disadvantage(s): sharding
-   You'll need to update your application logic to work with shards, which could result in complex SQL queries.
-   Data distribution can become lopsided in a shard. For example, a set of power users on a shard could result in increased load to that shard compared to others.
    -   Rebalancing adds additional complexity. A sharding function based on [consistent hashing](http://www.paperplanes.de/2011/12/9/the-magic-of-consistent-hashing.html) can reduce the amount of transferred data.
-   Joining data from multiple shards is more complex.
![[Pasted image 20220417103938.png]]

## Denormalization

Denormalization attempts to improve read performance at the expense of some write performance. Redundant copies of the data are written in multiple tables to avoid expensive joins. Some RDBMS such as [PostgreSQL](https://en.wikipedia.org/wiki/PostgreSQL) and Oracle support [materialized views](https://en.wikipedia.org/wiki/Materialized_view) which handle the work of storing redundant information and keeping redundant copies consistent.

Once data becomes distributed with techniques such as [federation](https://github.com/donnemartin/system-design-primer#federation) and [sharding](https://github.com/donnemartin/system-design-primer#sharding), managing joins across data centers further increases complexity. Denormalization might circumvent the need for such complex joins.

In most systems, reads can heavily outnumber writes 100:1 or even 1000:1. A read resulting in a complex database join can be very expensive, spending a significant amount of time on disk operations.

##### [](https://github.com/donnemartin/system-design-primer#disadvantages-denormalization)Disadvantage(s): denormalization
-   Data is duplicated.
-   Constraints can help redundant copies of information stay in sync, which increases complexity of the database design.
-   A denormalized database under heavy write load might perform worse than its normalized counterpart.


## NoSQL
NoSQL is a collection of data items represented in a **key-value store**, **document store**, **wide column store**, or a **graph database**. Data is denormalized, and joins are generally done in the application code. Most NoSQL stores lack true ACID transactions and favor [eventual consistency](https://github.com/donnemartin/system-design-primer#eventual-consistency).

#### Key-value store
Abstraction: hash table

A key-value store generally allows for O(1) reads and writes and is often backed by memory or SSD. Data stores can maintain keys in lexicographic order (aka alphabetical, totally ordered), allowing efficient retrieval of key ranges. Key-value stores can allow for storing of metadata with a value.

Key-value stores provide high performance and are often used for simple data models or for rapidly-changing data, such as an in-memory cache layer. Since they offer only a limited set of operations, complexity is shifted to the application layer if additional operations are needed.

#### Document store
Abstraction: key-value store with documents stored as values

A document store is centered around documents (XML, JSON, binary, etc), where a document stores all information for a given object. Document stores provide APIs or a query language to query based on the internal structure of the document itself. _Note, many key-value stores include features for working with a value's metadata, blurring the lines between these two storage types._

#### Wide column store
![[Pasted image 20220417112330.png]]

A wide column store's basic unit of data is a column (name/value pair). A column can be grouped in column families (analogous to a SQL table). Super column families further group column families

#### Graph database
![[Pasted image 20220417112417.png]]
In a graph database, each node is a record and each arc is a relationship between two nodes. Graph databases are optimized to represent complex relationships with many foreign keys or many-to-many relationships.

Graphs databases offer high performance for data models with complex relationships, such as a social network. They are relatively new and are not yet widely-used.

## SQL or NoSQL
[[Cliff Notes - moyano83#Relational Model Versus Document Model]]
[[Cliff Notes - Someguy#Relational model vs document model]]

### Reasons for **SQL**:
-   Structured data
-   Strict schema
-   Relational data
-   Need for complex joins
-   Transactions
-   Clear patterns for scaling
-   More established: developers, community, code, tools, etc
-   Lookups by index are very fast

### Reasons for **NoSQL**:
-   Semi-structured data
-   Dynamic or flexible schema
-   Non-relational data
-   No need for complex joins
-   Store many TB (or PB) of data
-   Very data intensive workload
-   Very high throughput for IOPS

Sample data well-suited for NoSQL:
-   Rapid ingest of clickstream and log data
-   Leaderboard or scoring data
-   Temporary data, such as a shopping cart
-   Frequently accessed ('hot') tables
-   Metadata/lookup tables

