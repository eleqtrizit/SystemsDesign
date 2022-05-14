## FEATURE EXPECTATIONS - 5 min
- Waht does it need to do?
- Use cases
- Scenarios that will not be covered
- Who will use
- How many will use
- Usage patterns

## ESTIMATIONS - 5 min
- Throughput (QPS for read and write queries)
- Latency expected from the system (for read and write queries)
- Read/Write ratio
- Traffic estimates
	- Write (QPS, Volume of data)
	- Read  (QPS, Volume of data)
- Storage estimates
- Memory estimates
	- If we are using a cache, what is the kind of data we want to store in cache
	- How much RAM and how many machines do we need for us to achieve this ?
	- Amount of data you want to store in disk/ssd

## DESIGN GOALS - 5 min
- Latency and Throughput requirements
- Consistency vs Availability  [Weak/strong/eventual => consistency | Failover/replication => availability]


## HIGH LEVEL DESIGN - 5-10 min
- APIs for Read/Write scenarios for crucial components
- Database schema
- Basic algorithm
- High level design for Read/Write scenario

## DEEP DIVE - 15-20 min
- Deployment
	- [[Google Systems Design - Code Deployment]]
- Scaling the algorithm
- Scaling individual components: 
	-  Availability, Consistency and Scale story for each component
	-  Consistency and availability patterns
- Call graph depth
	- Longer sequence call depths in the graph means compounding availability problems
		- If both `Foo` and `Bar` each had 99.9% availability, their total availability in sequence would be 99.8%.
-  What is a graceful (aka. zero downtime) restart?  
	- A graceful restart takes place in two steps:
		-   A new generation of parent and child processes are spawned to take over service.
		-   Processes from the older generation only exit once they have finished their tasks.
- Think about the following components, how they would fit in and how it would help
	- DNS [[Networking#How does DNS work]]
	-  CDN [Push vs Pull]
	-  Load Balancers [Active-Passive, Active-Active, Layer 4, Layer 7]
		- Reverse Proxy
	- **Single points of failure**
	-  Application layer scaling [Microservices, Service Discovery]
	-  DB [[Cliff Notes - Someguy#Relational model vs document model]]
		-  RDBMS 
			-  Master-slave, Master-master, Federation, Sharding, Denormalization, SQL Tuning
			- Benefits - great at many-to-many and many-to-one relationships, one copy of data
			- Downfalls - high impedance (results format don't match the model format)
		-  NoSQL
			- Benefits: Greater scalability, high write throughput, large datasets, dynamic schemas, low impendance
			- Downfalls: not good at many-to-one or many-to-many relationships, data can be replicated in many objects, eventual consistency
			-  Key-Value, Wide-Column, Graph, Document
				- Fast-lookups:
					-  RAM  [Bounded size] => Redis, Memcached
					-  AP [Unbounded size] => Cassandra, RIAK, Voldemort
					-  CP [Unbounded size] => HBase, MongoDB, Couchbase, DynamoDB
	-  **Caches**
		- [[Caching]]
	-  **Asynchronism**
		-  Message queues
		-  Task queues
		-  Back pressure
	-  **Communication**
		-  TCP
		-  UDP
		-  REST
		-  RPC
	- **Metrics**
		- External (off-box, not off prem) Monitoring
			- in case local monitoring collapses
			- regional problems e.g. check for down isps
			- breaches in SLA
			- QPS, Latency, Errors, Calltimes
		- Host 
			- health check
			- disk space, memory, swap, i/o, load
			- anything specific to the programming language
				- e.g. Java GC
		- SLA  Alerting/Reporting/Breaches
			- An SLA is a contract.
			-   An SLO is a specific goal that is defined in a contract.
			-   An SLI measures the extent to which teams comply with the SLO promises they make in SLA contracts.
				- The SLI (Service Level Indicator) equation is **the number of good events divided by the total number of valid events, multiplied by 100 to keep it a uniform percentage**.
				- measures how well a company actually meets the SLO promises that it sets within SLAs.
				- Measured over some period
			- Burn rate - notify when you're consuming your error budget faster than threshold, measured over compliance period
	- Log Aggregation
		- Why this is important in a high-availability / distributed system
		- Query-able


## JUSTIFY - 5 min
	-  Throughput of each layer
	-  Latency caused between each layer
	-  Overall latency justification
