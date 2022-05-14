# System Design Interview Steps

The system design interview is an **open-ended conversation**. You are expected to lead it.

You can use the following steps to guide the discussion. To help solidify this process, work through the [System design interview questions with solutions](https://github.com/donnemartin/system-design-primer#system-design-interview-questions-with-solutions) section using the following steps.

## Step 1: Outline use cases, constraints, and assumptions
-   Who is going to use it?
-   How are they going to use it?
-   How many users are there?
-   What does the system do?
-   What are the inputs and outputs of the system?
-   How much data do we expect to handle?
-   How many requests per second do we expect?
-   What is the expected read to write ratio?

## Step 2: Create a high level design

Outline a high level design with all important components.

-   Sketch the main components and connections
-   Justify your ideas

## Step 3: Design core components

Dive into details for each core component. For example, if you were asked to [design a url shortening service](https://github.com/donnemartin/system-design-primer/blob/master/solutions/system_design/pastebin/README.md), discuss:

-   Generating and storing a hash of the full url
    -   MD5/Base64
    -   Hash collisions
    -   SQL or NoSQL
    -   Database schema
-   Translating a hashed url to the full url
    -   Database lookup
-   API and object-oriented design

## Step 4: Scale the design

Identify and address bottlenecks, given the constraints. For example, do you need the following to address scalability issues?

-   Load balancer
-   Horizontal scaling
-   Caching
-   Database sharding

Discuss potential solutions and trade-offs. Everything is a trade-off. Address bottlenecks using [principles of scalable system design](https://github.com/donnemartin/system-design-primer#index-of-system-design-topics).

## Source(s) and further reading

Check out the following links to get a better idea of what to expect:

-   [How to ace a systems design interview](https://www.palantir.com/2011/10/how-to-rock-a-systems-design-interview/)
-   [The system design interview](http://www.hiredintech.com/system-design)
-   [Intro to Architecture and Systems Design Interviews](https://www.youtube.com/watch?v=ZgdS0EUmn70)
-   [System design template](https://leetcode.com/discuss/career/229177/My-System-Design-Template)


# Back of the Envelope Calculations

## Numbers Everyone Should Know

To evaluate design alternatives you first need a good sense of how long typical operations will take. Dr. Dean gives this list:
```
Latency Comparison Numbers
--------------------------
L1 cache reference                           0.5 ns
Branch mispredict                            5   ns
L2 cache reference                           7   ns                      14x L1 cache
Mutex lock/unlock                           25   ns
Main memory reference                      100   ns                      20x L2 cache, 200x L1 cache
Compress 1K bytes with Zippy            10,000   ns       10 us
Send 1 KB bytes over 1 Gbps network     10,000   ns       10 us
Read 4 KB randomly from SSD*           150,000   ns      150 us          ~1GB/sec SSD
Read 1 MB sequentially from memory     250,000   ns      250 us
Round trip within same datacenter      500,000   ns      500 us
Read 1 MB sequentially from SSD*     1,000,000   ns    1,000 us    1 ms  ~1GB/sec SSD, 4X memory
HDD seek                            10,000,000   ns   10,000 us   10 ms  20x datacenter roundtrip
Read 1 MB sequentially from 1 Gbps  10,000,000   ns   10,000 us   10 ms  40x memory, 10X SSD
Read 1 MB sequentially from HDD     30,000,000   ns   30,000 us   30 ms 120x memory, 30X SSD
Send packet CA->Netherlands->CA    150,000,000   ns  150,000 us  150 ms

Notes
-----
1 ns = 10^-9 seconds
1 us = 10^-6 seconds = 1,000 ns
1 ms = 10^-3 seconds = 1,000 us = 1,000,000 ns
```

## Design 1 - Serial 
-   Read images serially. Do a disk seek. Read a 256K image and then go on to the next image.
-   Performance: 30 seeks * 10 ms/seek + 30 * 256K / 30 MB /s = 560ms

## Design 2 - Parallel 
-   Issue reads in parallel.
-   Performance: 10 ms/seek + 256K read / 30 MB/s = 18ms
-   There will be variance from the disk reads, so the more likely time is 30-60ms

Which design is best? It depends on you requirements, but given the back-of-the-envelope calculations you have a quick way to compare them without building them.

Now you have a framework for asking yourself other design questions and comparing different design variations:
-   Does it make sense to cache single thumbnail images?
-   Should you cache a whole set of images in one entry?
-   Does it make sense to precompute the thumbnails?


# System Design Topics

## Step 1: Review the scalability video lecture

[Scalability Lecture at Harvard](https://www.youtube.com/watch?v=-W9F__D3oY4)

-   Topics covered:
    -   Vertical scaling
    -   Horizontal scaling
    -   Caching
    -   Load balancing
    -   Database replication
    -   Database partitioning

## Step 2: Review the scalability article

[Scalability](http://www.lecloud.net/tagged/scalability/chrono)

-   Topics covered:
    -   [Clones](http://www.lecloud.net/post/7295452622/scalability-for-dummies-part-1-clones)
    -   [Databases](http://www.lecloud.net/post/7994751381/scalability-for-dummies-part-2-database)
    -   [Caches](http://www.lecloud.net/post/9246290032/scalability-for-dummies-part-3-cache)
    -   [Asynchronism](http://www.lecloud.net/post/9699762917/scalability-for-dummies-part-4-asynchronism)


# Performance vs Scalability

A service is **scalable** if it results in increased **performance** in a manner proportional to resources added. Generally, increasing performance means serving more units of work, but it can also be to handle larger units of work, such as when datasets grow.[1](http://www.allthingsdistributed.com/2006/03/a_word_on_scalability.html)

Another way to look at performance vs scalability:
-   If you have a **performance** problem, your system is slow for a single user.
-   If you have a **scalability** problem, your system is fast for a single user but slow under heavy load.

## Latency vs Throughput

**Latency** is the time to perform some action or to produce some result.
- e.g. requests take 100ms

**Throughput** is the number of such actions or results per unit of time.
* e.g. The system can handle 10,000 requests per hour


## Availability vs Consistency

More: [[CAP Theorum]]


### Replication
#### Master-slave and master-master
[[Databases]]

