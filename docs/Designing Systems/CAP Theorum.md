# CAP Theorum

  
![[Pasted image 20220411202403.png]]
  
![[Pasted image 20220411194559.png]]

In a distributed computer system, you can only support two of the following guarantees:
-   **Consistency** - Data is always up-to-date on every read.  Every read receives the most recent write or an error.
-   **Availability** - Data is always accessible.  Every request receives a response, **without guarantee** that it contains the most recent version of the information
-   **Partition Tolerance** -  the distributed system continues to operate in the presence of a failure of the network where not all members can reach other members.

## Important Note
**_Networks aren't reliable, so you'll need to support partition tolerance. You'll need to make a software tradeoff between consistency and availability._**

### CP - consistency and partition tolerance

Waiting for a response from the partitioned node might result in a timeout error. CP is a good choice if your business needs require atomic reads and writes.

### AP - availability and partition tolerance

Responses return the most readily available version of the data available on any node, which might not be the latest. Writes might take some time to propagate when the partition is resolved.

AP is a good choice if the business needs allow for [eventual consistency](https://github.com/donnemartin/system-design-primer#eventual-consistency) or when the system needs to continue working despite external errors.


## Consistency patterns

With multiple copies of the same data, we are faced with options on how to synchronize them so clients have a consistent view of the data. Recall the definition of consistency from the [CAP theorem](https://github.com/donnemartin/system-design-primer#cap-theorem) - Every read receives the most recent write or an error.

### Weak consistency

After a write, reads may or may not see it. A best effort approach is taken.

This approach is seen in systems such as memcached. Weak consistency works well in real time use cases such as VoIP, video chat, and realtime multiplayer games. For example, if you are on a phone call and lose reception for a few seconds, when you regain connection you do not hear what was spoken during connection loss.

### Eventual consistency

After a write, reads will eventually see it (typically within milliseconds). Data is replicated asynchronously.

This approach is seen in systems such as DNS and email. Eventual consistency works well in highly available systems.

### Strong consistency

After a write, reads will see it. Data is replicated synchronously.

This approach is seen in file systems and RDBMSes. Strong consistency works well in systems that need transactions.

## Availability patterns
### Fail-over
#### Active-passive
With active-passive fail-over, heartbeats are sent between the active and the passive server on standby. If the heartbeat is interrupted, the passive server takes over the active's IP address and resumes service.

The length of downtime is determined by whether the passive server is already running in 'hot' standby or whether it needs to start up from 'cold' standby. Only the active server handles traffic.

Active-passive failover can also be referred to as master-slave failover.

#### Active-active
In active-active, both servers are managing traffic, spreading the load between them.

If the servers are public-facing, the DNS would need to know about the public IPs of both servers. If the servers are internal-facing, application logic would need to know about both servers.

Active-active failover can also be referred to as master-master failover.

### Disadvantage(s): failover
-   Fail-over adds more hardware and additional complexity.
-   There is a potential for loss of data if the active system fails before any newly written data can be replicated to the passive.


## Databases
### CA databases
CA databases enable consistency and availability across all nodes. Unfortunately, CA databases **can’t deliver fault tolerance**. In any distributed system, partitions are bound to happen, which means this type of database isn’t a very practical choice. That being said, you still can find a CA database if you need one. Some [**relational databases**](https://www.educative.io/blog/relational-database-deep-dive), such as [PostgreSQL](https://www.educative.io/blog/mongodb-versus-postgresql-databases), allow for consistency and availability. You can deploy them to nodes using replication.

### CP databases
CP databases enable consistency and partition tolerance, but not availability. When a partition occurs, the system has to **turn off inconsistent nodes until the partition can be fixed**. [MongoDB](https://www.educative.io/blog/mongodb-with-docker) is an example of a CP database. It’s a NoSQL database management system (DBMS) that uses documents for data storage. It’s considered schema-less, which means that it doesn’t require a defined database schema. It’s commonly used in [big data](https://www.educative.io/blog/what-is-big-data) and applications running in different locations. The CP system is structured so that there’s only one primary node that receives all of the write requests in a given replica set. Secondary nodes replicate the data in the primary nodes, so if the primary node fails, a secondary node can stand-in.

### AP databases
AP databases enable availability and partition tolerance, but not consistency. In the event of a partition, all nodes are available, but they’re not all updated. For example, if a user tries to access data from a bad node, they **won’t receive the most up-to-date version of the data**. When the partition is eventually resolved, most AP databases will sync the nodes to ensure consistency across them. **Apache Cassandra** is an example of an AP database. It’s a NoSQL database with no primary node, meaning that all of the nodes remain available. Cassandra allows for eventual consistency because users can resync their data right after a partition is resolved.