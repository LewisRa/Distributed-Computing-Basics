# Distributed Systems Basics
A distributed system in its most simplest definition is a group of computers working together as to appear as a single computer to the end-user.

Traditional databases are stored on the **filesystem** of one single machine, whenever you want to fetch/insert information in it — you talk to that machine directly but with distributed systems, all cluster machines have a **shared state**, operate **concurrently** and can fail independently without affecting the whole system’s uptime. EX: If a user inserts a record into node #1, node #3 must be able to return that record.

Distributed systems are a headache to deploy, maintain and debug distributed systems,but it enables you to scale **horizontally**, meaning you add more computers rather than upgrading the hardware of a single one **vertically**.

**Easy scaling is not the only benefit you get from distributed systems. Fault tolerance and low latency are also equally as important.**

**Fault Tolerance** — a cluster of ten machines across two data centers is inherently more fault-tolerant than a single machine. Even if one data center catches on fire, your application would still work.

**Low Latency** — The time for a network packet to travel the world is physically bounded by the speed of light. For example, the shortest possible time for a request‘s round-trip time (that is, go back and forth) in a fiber-optic cable between New York to Sydney is 160ms. Distributed systems allow you to have a node in both cities, allowing traffic to hit the node that is closest to it.

For a distributed system to work, though, you need the software running on those machines to be specifically designed for running on multiple computers at the same time and handling the problems that come along with it. This turns out to be no easy feat.

---
## Ex: Scaling a database
In a typical web application you normally read information much more frequently than you insert new information or modify old one.

#### Master-Slave
There is a way to increase read performance and that is by the so-called Master-Slave Replication strategy. Here, you create two new database servers which sync up with the main one. The catch is that you can only read from these new instances.Whenever you insert or modify information — you talk to the master database. It, in turn, asynchronously informs the slaves of the change and they save it as well. But you lost the 'C' in ACID: consistency because there now exists a possibility in which we insert a new record into the database, immediately afterwards issue a read query for it and get nothing back.

Propagating the new information from the master to the slave does not happen instantaneously. There actually exists a time window in which you can fetch stale information. If this were not the case, your write performance would suffer, as it would have to synchronously wait for the data to be propagated.

Using the slave database approach, we can horizontally scale our read traffic up to some extent. That’s great but we’ve hit a wall in regards to our write traffic — it’s still all in one server. ----->  **multi-master replication strategy** -With this, instead of slaves that you can only read from, you have multiple master nodes which support reads and writes. Unfortunately, this gets complicated real quick as you now have the ability to create conflicts (e.g insert two records with same ID).

#### Sharding (Partitioning)(best avoided until really needed because SQL join are overly complex)

With sharding you split your server into multiple smaller servers, called shards. These shards all hold different records — you create a rule as to what kind of records go into which shard. It is very important to create the rule such that the data gets spread in an uniform way. A possible approach to this is to define ranges according to some information about a record (e.g users with name A-D).
**BUT if a single shard receives more requests than others is called a hot spot and must be avoided. Once split up, re-sharding data becomes incredibly expensive and can cause significant downtime, as was the case with FourSquare’s infamous 11 hour outage** 

---
## 1. Distributed Data Stores
Most distributed databases are NoSQL non-relational databases, limited to key-value semantics.

**Cassandra** <br>
Cassandra is a distributed No-SQL database which prefers the AP properties out of the CAP, settling with eventual consistency. It uses consistent hashing to determine which nodes out of your cluster must manage the data you are passing in. You set a replication factor, which basically states to how many nodes you want to replicate your data.
(Cassandra, Couchbase, Hbase, MongoDB)

Apple is known to use 75,000 Apache Cassandra nodes storing over 10 petabytes of data, back in 2015

#### Consensus
Database transactions are tricky to implement in distributed systems as they require each node to agree on the right action to take (abort or commit). This is known as consensus and it is a fundamental problem in distributed systems. For example, cassandra actually provides lightweight transactions through the use of the Paxos algorithm for distributed consensus.

## 2. Distributed Computing
**Distributed Computing falls in two broad categories: Master/Slave Node vs Peer to Peer**?
Distributed Computing is the technique of splitting an enormous task (e.g aggregate 100 billion records), of which no single computer is capable of practically executing on its own, into many smaller tasks, each of which can fit into a single commodity machine. You split your huge task into many smaller ones, have them execute on many machines in parallel, aggregate the data appropriately and you have solved your initial problem. This approach again enables you to scale horizontally — when you have a bigger task, simply include more nodes in the calculation.

--MapReduce <br>
--Lambda <br>
--Kappa <br>
#### Distributed  Computing Example: Google
- current storage = 15 exabytes
- Processed per day = 100 petabytes
- number of pages indexed = 60 trillion
- unique seatch users per month > 1 billion
- searches by seconds = 2.3 million

## 3. Distributed File Systems
Distributed file systems can be thought of as distributed data stores. They’re the same thing as a concept — storing and accessing a large amount of data across a cluster of machines all appearing as one. They typically go hand in hand with Distributed Computing. **Wikipedia defines the difference being that distributed file systems allow files to be accessed using the same interfaces and semantics as local files, not through a custom API like the Cassandra Query Language (CQL).**

--HDFS

## 4. Distributed Messaging
Messaging systems provide a central place for storage and propagation of messages/events inside your overall system. They allow you to decouple your application logic from directly talking with your other systems.

A message is broadcast from the application which potentially create it (called a producer), goes into the platform and is read by potentially multiple applications which are interested in it (called consumers).Consumers can either pull information out of the brokers (pull model) or have the brokers push information directly into the consumers (push model).

--Kafka <br>
--RabbitMQ <br>
--Apache ActiveMQ <br>
--Amazon SQS (AWS) <br>
 
 ## 5. Distributed Applications (Erlang Machine Machine,BitTorrent)
 BitTorrent is one of the most widely used protocol for transferring large files across the web via torrents(peer to peer). The main idea is to facilitate file transfer between different peers in the network without having to go through a main server.
 Using a BitTorrent client, you connect to multiple computers across the world to download a file. When you open a .torrent file, you connect to a so-called tracker, which is a machine that acts as a coordinator. 
 
 You have the notions of two types of user, a leecher and a seeder. A leecher is the user who is downloading a file and a seeder is the user who is uploading said file.Freeriding, where a user would only download files, was an issue with the previous file sharing protocols.
 
 ## 6. Distributed Ledgers (Blockchain, Ethereum)
 ---
 https://www.freecodecamp.org/news/a-thorough-introduction-to-distributed-systems-3b91562c9b3c/
