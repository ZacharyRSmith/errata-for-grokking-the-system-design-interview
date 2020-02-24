# Errata & Clarifications for "Grokking the System Design Interview"

"Grokking the System Design Interview" is a popular study guide.  Unfortunately at least 4 of its pages have many errors.  This provides errata & clarifications on these 4 pages.  It is as much notes for myself as for others, so my apologies for any unclear language.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Data Partitioning](#data-partitioning)
  - [Section 1. Partitioning Methods](#section-1-partitioning-methods)
  - [Section 2. Partitioning Criteria](#section-2-partitioning-criteria)
  - [Section 3. Common Problems of Data Partitioning](#section-3-common-problems-of-data-partitioning)
- [SQL vs. NoSQL](#sql-vs-nosql)
- [CAP Theorem](#cap-theorem)
- [Redundancy and Replication](#redundancy-and-replication)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


# Data Partitioning

General correction: This page speaks as if nodes have a 1-to-1 relationship with partitions. This is false: nodes usually have a 1-to-many relationship with partitions, and sometimes have 0 partitions. This enables moving entire partitions between nodes, rather than the less efficient moving of data between partitions.

General clarification: This page speaks as if partitioning is usually implemented without replication.  This is false: partitioning is usually combined with replication.

## Section 1. Partitioning Methods

Correction: "c. Directory Based Partitioning" is not in the same group as "a. Horizontal partitioning" and "b. Vertical Partitioning". Instead, it is in the group of "request routing" methods, aka service discovery. There are 3 main approaches to request routing for partitions. They differ on who knows how to route requests:
- clients (eg, your application needs to know how to access nodes)
- nodes (nodes forward requests to relevant nodes)
- routing tier (eg, ZooKeeper)

## Section 2. Partitioning Criteria

Clarification: This section fails to present a criteria more common than list or round-robin partitioning: partitioning by range of key.

Here's an example of partitiong by range of key: you want to partition User records by last_name. So, records whose last_name is in range `'A' <= last_name < 'B'` go to partition A, records in range `'B' <= last_name < 'C'` go to partition B, etc. But this causes imbalance across partitions, so maybe records in range `'X' <= last_name` go to 1 partition, and S is split into 2 ranges/partitions: `'S' <= last_name < 'Smith'` and `'Smith' <= last_name < 'T'`.

Correction: The example in the "a. Key or Hash-based partitioning" paragraph is said to require downtime for the service, even though databases can migrate data while serving requests. Also, it is imprecise to call this "key partitioning" (because hash, range, and list partitioning all partition by key). Also, it is potentially misleading by not providing middle ground between the terrible example hash function and consistent hashing. Here is a better quick description:

Partitioning by hash of key is a variation of partitioning by range of key: each partition is assigned a range(s) of possible hash values. Each range's size can be uniform, or pseudorandomly picked (aka "consistent hashing").

You want to assign each partition a range(s) of possible hash values rather than determining which partition a key belongs to by `partition_idx = hash(key) modulo num_partitions`. If you use the modulo approach, then when num_partitions change, most keys would need to be migrated.

Hash pros & cons:
pro: requires less rebalancing than range, and supports pseudorandomly picking partition ranges (aka "consistent hashing").
con: does not support range queries

## Section 3. Common Problems of Data Partitioning

Correction: "rebalance existing partitions, which means the partitioning scheme changed and all existing data moved to new locations" is false. Good databases do not move all existing data to new locations. Eg, HBase rebalances when a single partition becomes either too small (it is merged with an adjacent partition) or too big (it is split into 2 partitions). Note also that this might occur on the same node.

Correction: "Doing this without incurring downtime is extremely difficult" is false: most databases perform rebalancing while maintaining good performance.

Correction: "Using a [routing tier like ZooKeeper]...[creates] a new single point of failure" is false: routing tiers are usually implemented as a cluster, so they are not a single point of failure in the sense that this book uses.


# SQL vs. NoSQL

General correction: this page thinks that "columnar databases" are the same thing as "wide column databases", and they are not.  Ignore everything said about these types of databases, and learn about them elsewhere.

Correction: "Non-relational databases are...distributed" is false: not all NoSQL databases are distributed.

Clarification: "The schema [of an RDBMS] can be altered later, but it involves modifying the whole database and going offline" is misleading: most RDBMSes execute ALTER TABLE statements in a few milliseconds, which is technically but not practically "offline". An exception is MySQL: it copies the entire table, which can take hours.

Clarifications on the "Scalability" section:

Unlike what this page implies, many SQL and NoSQL databases can do all of these:
- vertically scale
- horizontally scale (although the relational model can make this harder)
- "hostable by cheap commodity hardware or cloud instances"
- distribute data across servers automatically

Clarification on "ACID compliance reduces anomalies and protects the integrity of your database": ACID compliance is neither necessary nor sufficient to protect the integrity of your database.

Clarification: The reasons given to use SQL or NoSQL databases is hardly exhaustive. Eg, here is a reason to choose NoSQL that surprises many people: in some cases, a NoSQL database increases data consistency as compared to a SQL database.


# CAP Theorem

This page misunderstands the CAP theorem. It would be better to completely ignore and find some other resource on the CAP theorem. However, for completeness here are some corrections:

Correction: "Consistency is achieved by updating several nodes before allowing further reads" is false: consistency can mean many different things, and in the context of the CAP theorem it means linearizability, which cannot be achieved merely by "updating several nodes before allowing further reads".

Correction: "Availability: Every request gets a response on success/failure. Availability is achieved by replicating the data across different servers" is false: In the context of the CAP theorem, availability means "total availability", i.e. every node responds successfully. Also, replication actually makes total availability _more_ difficult.

Correction: "Partition tolerance: The system continues to work despite message loss or partial failure. A system that is partition-tolerant can sustain any amount of network failure that doesn’t result in a failure of the entire network. Data is sufficiently replicated across combinations of nodes and networks to keep the system up through intermittent outages" is false: this is not the CAP theorem's definition of partition tolerance. The wording of this paragraph makes it difficult to see, but it claims that all data should be available from each node. Wikipedia's is accurate: "The system continues to operate despite an arbitrary number of messages being dropped (or delayed) by the network between nodes".

Correction on the image: "Availability: System continues to function even with node failures" is false (see above). The CAP theorem applies only to network partitions, not to node failures or any other fault.

Correction on the image: RDBMSes often claim to have a config for sync replication, which would guarantee CAP-consistency at the cost of CAP-availability during CAP-partitions, but it's usually actually async replication w/1 sync follower for durability, which sacrifices CAP-consistency (even without a CAP-partition) to increase availability.

Correction on the image: Almost all instances of BigTable, MongoDB, and HBase are not CAP-consistent.

Correction on the image: Almost all instances of Cassandra (and probably CouchDB, but I'm not familiar with it) are not CAP-available.


# Redundancy and Replication


Correction: This page speaks as if there is only one method to replicate: "The master gets all the updates, which then ripple through to the slaves. Each slave outputs a message stating that it has received the update successfully, thus allowing the sending of subsequent updates." This is false: there are other methods. Eg, the master's replication log could have a counter that orders writes, and these logs are sent ASAP to slaves. Some happened-later logs might arrive at a slave before happened-earlier logs, but the slave knows to not apply those logs because their counter is not the next count to apply.
