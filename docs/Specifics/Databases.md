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

## Synchronous vs Asynchronous Commits
### Asynchronous Commit
In an asynchronous commit, the transaction is considered complete as soon as the command is issued, without waiting for the data to be physically written to the disk. This approach can improve performance, especially in situations involving a large number of write operations.

However, there's a risk involved: if a system failure occurs before the data is physically written to the disk, the changes may be lost.

**Use Cases for Asynchronous Commit**
- When high throughput and lower latency are more important than the risk of potential data loss.
- When the data is not mission-critical and can be recovered or recreated if lost.
- When the database is used for caching data or for temporary storage.

### Synchronous Commit
In a synchronous commit, the transaction is not considered complete until the data has been physically written to the disk. This ensures a higher level of data integrity as it reduces the risk of data loss in case of a system failure.

However, this approach may have a performance impact as it requires the system to wait until the write operation is confirmed.

**Use Cases for Synchronous Commit**
- When data integrity is paramount and any data loss is unacceptable.
- When the database stores critical data, such as financial transactions, where consistency and durability are required.
- When the system can tolerate the potential performance impact of waiting for commit confirmation.
- In summary, the decision to use asynchronous or synchronous commit depends largely on the trade-off between performance and data integrity that is acceptable for your specific use case.

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


## Change Control
### Database Content Control
Change control for iterative structure releases in a database can be achieved through a combination of version control systems, migration scripts, and database schema management tools.

#### Version Control Systems
Version Control Systems (like Git) can be used to keep track of changes made to the database structure. Each change to the database structure is committed to the version control system with a description of what was changed, why it was changed, and by whom. This allows for easy tracking and rollback of changes if necessary.

#### Migration Scripts
Migration scripts are scripts that make changes to the database structure. These scripts are written in such a way that they can be run multiple times without causing issues. Each script corresponds to a specific change to the database structure.

#### Database Schema Management Tools
Database Schema Management Tools can be used to manage and automate the process of applying migration scripts. They keep track of which scripts have been run, ensuring that each script is only run once.

### Database Configuration Control

This section is weak.  I'm guessing it is a combination of documentation, config management repo, a tool to deploy the changes, and a tool to test/benchmark the changes before a full roll out versus the existing configuration.

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


### Popular NoSQL Options
There are several major non-relational database options, each with its own unique strengths. Here are some of the most popular ones:


#### MongoDB
**Strengths**: Flexible schema, scalability, and speed.
MongoDB uses a document-oriented model which allows for records that do not have to have a uniform structure, unlike relational databases. This flexibility can make it easier to scale and develop with.

#### Cassandra
**Strengths**: High availability, fault tolerance, and performance.
Cassandra was designed to handle large amounts of data across many commodity servers, providing high availability with no single point of failure.

#### Redis
**Strengths**: Speed, simplicity, and versatility.
Redis is an in-memory data structure store used as a database, cache, and message broker. Its in-memory nature gives it excellent performance.

#### HBase
**Strengths**: Scalability, strong consistency, and integration with Hadoop.
HBase is a column-oriented database that's designed to store large amounts of data in a distributed environment. It's built on top of Hadoop and is often used for real-time read/write access to big data.

#### CouchDB
**Strengths**: Replication & Sync, ease of use, and a HTTP/JSON API.

CouchDB is a document-oriented database and uses JSON to store data, JavaScript as its query language, and HTTP for an API. It also offers incremental replication with bi-directional conflict detection and resolution.


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

## Testing

Strategies would aim to maintain data integrity, database performance, and continuity of operations even under unexpected conditions.

**Implement Data Validation Rules**: Use database constraints (such as UNIQUE, NOT NULL, CHECK) to ensure data consistency and prevent invalid data from being inserted into the database.
```sql
CREATE TABLE Employees (
 ID int NOT NULL,
 Name varchar(255) NOT NULL,
 Age int CHECK (Age>=18),
 PRIMARY KEY (ID)
);
```

**Use Exception Handling**: Implement error handling mechanisms in your SQL procedures to catch exceptions and manage them appropriately.
```sql
BEGIN TRY
 -- Code here
END TRY
BEGIN CATCH
 -- Handle error
END CATCH
```

**Write Unit Tests**: Create tests for your database procedures and queries to check their correctness under a variety of scenarios, including corner cases.
```sql
-- A simple unit test might look like this (pseudo-code)
TEST PROCEDURE test_employee_age() AS
BEGIN
 -- Insert an employee with an invalid age
 INSERT INTO Employees (ID, Name, Age) VALUES (1, 'Test', 17);
 -- Check that the insert failed
 ASSERT SELECT COUNT(*) FROM Employees WHERE ID = 1 IS NULL;
END
```

**Perform Stress Testing**: Test how your database performs under heavy load or large amounts of data to identify and address performance issues.

**Design for Scalability**: Use strategies like partitioning, sharding, or using distributed databases to ensure your database can handle high volumes of data.

**Regularly Monitor and Profile Your Database**: Use database profiling tools to regularly monitor the performance of your database and identify any potential corner cases causing issues.

**Use Transaction Management**: Use transactions to ensure that your database operations are atomic, consistent, isolated, and durable (ACID). This is especially important for handling corner cases where operations might fail partway.

## Security

Database security is a critical aspect of information system design and administration. There are two primary types of privileges in a database environment: administrative privileges and system privileges. Here's a detailed breakdown of the two and their associated security concerns:

### Administrative Privileges
Administrative privileges allow a user to perform high-level operations in a database. These operations include:

- Creating, altering, and dropping databases and tables
- Granting, revoking, and altering user privileges
- Running backup and restore operations

#### Security Concerns
**Access Control**: The primary security concern is unauthorized access. If a malicious user gains administrative access, they could potentially alter the database, damage the data, or grant themselves further permissions.

**Data Integrity**: Administrators have the ability to alter the database schema. If not properly controlled, this could lead to issues with data integrity.

**Data Loss**: Administrators typically have the ability to drop databases and tables. If a malicious user or even a careless administrator were to drop critical databases or tables, it could result in significant data loss.

### System Privileges
System privileges are more general than administrative privileges and allow a user to perform actions within the operating system that the database resides on. These privileges include:

- Reading and writing files
- Starting and stopping services
- Installing and uninstalling software

### Security Concerns
**System Integrity**: If a user has system privileges, they can potentially alter critical system settings, which could destabilize the system or make it vulnerable to further attacks.

**Data Privacy**: System privileges often give the user the ability to read and write files. This could potentially be used to access sensitive data or introduce malicious software.

**Denial of Service**: A user with system privileges could potentially stop critical services, resulting in a denial of service.

To mitigate these security concerns, it's important to follow the **principle of least privilege**, *which means only granting users the minimum privileges they need to perform their tasks*. Additionally, regular audits and monitoring can help detect any unusual activity or privilege escalation.

 