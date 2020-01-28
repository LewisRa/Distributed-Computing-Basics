# Distributed-Computing-Basics

 A distributed system in its most simplest definition is a group of computers working together as to appear as a single computer to the end-user.

These machines have a **shared state**, operate **concurrently** and can fail independently without affecting the whole system’s uptime.
if a user inserts a record into node#1, node #3 must be able to return that record.

Traditional databases are stored on the **filesystem** of one single machine, whenever you want to fetch/insert information in it — you talk to that machine directly.

Distributed systems are a headache to deploy, maintain and debug distributed systems,but it enables you to scale hoirzonatlly, meaning
you add more computers rather than upgrading the hardware of a single one.

**Easy scaling is not the only benefit you get from distributed systems. Fault tolerance and low latency are also equally as important.**

**Fault Tolerance** — a cluster of ten machines across two data centers is inherently more fault-tolerant than a single machine. Even if one data center catches on fire, your application would still work.

**Low Latency** — The time for a network packet to travel the world is physically bounded by the speed of light. For example, the shortest possible time for a request‘s round-trip time (that is, go back and forth) in a fiber-optic cable between New York to Sydney is 160ms. Distributed systems allow you to have a node in both cities, allowing traffic to hit the node that is closest to it.

For a distributed system to work, though, you need the software running on those machines to be specifically designed for running on multiple computers at the same time and handling the problems that come along with it. This turns out to be no easy feat.
## Scaling our database

In a typical web application you normally read information much more frequently than you insert new information or modify old one.

#### Master-Slave
There is a way to increase read performance and that is by the so-called Master-Slave Replication strategy. Here, you create two new database servers which sync up with the main one. The catch is that you can only read from these new instances.Whenever you insert or modify information — you talk to the master database. It, in turn, asynchronously informs the slaves of the change and they save it as well. But you lost the 'C' in ACID: consistency because there now exists a possibility in which we insert a new record into the database, immediately afterwards issue a read query for it and get nothing back.

Propagating the new information from the master to the slave does not happen instantaneously. There actually exists a time window in which you can fetch stale information. If this were not the case, your write performance would suffer, as it would have to synchronously wait for the data to be propagated.

Using the slave database approach, we can horizontally scale our read traffic up to some extent. That’s great but we’ve hit a wall in regards to our write traffic — it’s still all in one server. ----->  **multi-master replication strategy** -With this, instead of slaves that you can only read from, you have multiple master nodes which support reads and writes. Unfortunately, this gets complicated real quick as you now have the ability to create conflicts (e.g insert two records with same ID).
#### Distributed Data Stores
Most distributed databases are NoSQL non-relational databases, limited to key-value semantics.

 Apple is known to use 75,000 Apache Cassandra nodes storing over 10 petabytes of data, back in 2015
---
## Distrbuted Computing Example: Google
- current storage = 15 exabytes
- Processed per day = 100 petabytes
- number of pages indexed = 60 trillion
- unique seatch users per month > 1 billion
- searches by seconds = 2.3 million

## Master/Slave Node vs Peer to Peer
Distributed Computing falls in two broad categories:


## Synchronous vs Asynchronous

### Synchronous
The most common example is the standard **blocking HTTP call.**

When you hit an API endpoint, or go to a webpage, you usually make a blocking HTTP to some resource out on the internet.

EXAMPLE:
1. Client Makes Request and waits (Client Process)
2. Service gets request and performs some function(s) (Server Process)
3. Server sends a response (Server Process)
4. Client gets response, resumes where left off (Client Process)

**This is BLOCKING from the POV client**

Once you make that request ,you will not move forward until that request comes back. On the backend the server could process the request many different ways taht could be fast or slow. But it needs to send that esponse back to unblock the client. 

If too much time passes the server may timeout or the client my close the request.

**Synchronous is time consuming but you do need it sometimes!** Ex2 : Transferring money. 

### Asynchronous
EXAMPLE: **Amazon.com**

What happens when you click the **Confirm Purchase** button on Amazon?

**What do you physically see?**
- Your computer gets redirected to a succes screen that: <br>
"Thanks for you purchase, please shop with us again!"<br>
AND you can countinue shopping. You are not blocked

**What you do not see?**
- Amazon's **payment** system needs to kick in to withdraw from your bank account
- Amazon's **inventory** system needs to kick in track the count of the item you just brought
- Amazon's **analytics** system gets an update with your purchase stats, machine learning, machine learning models are retrained, your recommendations are improved  etc.
- Amazon's **dispatch** has to be notified to make sure the warehouse is ready to deliver the item within two days.
- Etc, etc. 

**Amazon Summary**<br>
This was a very asynchronous experience. You kept moving forward to shop while other thing happened in the background.

#### 1. Messaging Queueing
Message queueing is a common way that important notifications and message are sent throughout a large system

Let's use our Amazon example: 

The momentt we press **Confirm Purchase**, a special message &p payload is generated andpuy into a Queue for processing.

Imagine something like this: 
``` 
{
Type: "PurchasedItemOnWebsite",
ItemId: 2341083, 
Date: 03/12/20,
UserId: "David Xiang"
Price: $323.49
Prime: NO, 
Instrutions: "Please put a ribbon on my package"
}
```

Other systems in Amazon have the abaility to **observe** that message queue

When this payload is broadcastedm the Payment, Analytics Inventory etc. will kick in: "Oh, Rachel just purchased someting, let me kick off my special processes to handle that."

This general style of communaication is called **Publish/Subscriber** and is a ""broadcast-style" of communication **Apache Kafka**

#### 2. WEBHOOKS

Example 1: Callbacks - Think of thses as Callbacks between computers.

- "Execute this thing first, then call this function when it's done." 
- "Execute this HTTP request, then call my success() if nothing broke."
- "Execute this HTTP request, then call my fail() if something broke."

**Webhooks** are essentially between computers.

Another Example: **Syncing with Salesforce**

You decide to integrat with Salesforce. You want to keep a copy of important Salesforce data onyour system to track some analytics with you data.

But what if the data changes every minute?
What if you have 1000 employees updating SF data.

Salesforce is in the cloud somewhere, outside your syste, how can you keep this sync?

**Via WEBHOOKS**

You tell Salesforce: <br>
"Hey, any time data changes on your system, I want you to call this special webhook(callback0 ehich hits my system." 

1. An employee logs into SF and changes a user's email address
2. SF calls tyou registerd webhookL
https://mysuperapp.Rach.com/api/v23/sf_user?payload...
2. The application(MySuperApp) recieves the call and changes email address in the company database too. 
---
