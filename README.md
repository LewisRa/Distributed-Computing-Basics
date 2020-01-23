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
# Patterns for Building Distributed Systems for The Enterprise
### Patterns
- CQRS
- Event Sourcing
- Domain Driven Design
- Event Driven Architectures
### Theory
-CAP Theorem - states that we have to forfeit either consistency or availability in the face of a **network partition**
- Eventual Consistency
- Full and Partial Order
**Practice building software using services, queues, Enterprise Service Buses, sagas and histories**

#### Example: Build distributed system that allows customers to redeem gift cards at restaurants. 

Imagine you go to a restaurant or a coffee shop and buy a gift card, and then you can use it at any location in the same chain to pay for food or drinks. And when the card runs low, you can simply add more money to it.

One of the really cool attributes of the distributed system built is that it could cache information at each location. So, when you went to use your card, the point of sales system would already know your balance. It would send the purchase transaction up to the central server, which would then replicate it back down to all the other stores so each store could authorize your purchase without connecting to the central server. It could sync up later. 

This caching feature sped up the line resulting in higher revenues for coffee shop, and it made the system much more reliable. Each store location was connecting to the central server over the internet using **commodity hardware and consumer ISPs**. If we could get a customer on their way without making a connection every time, then the store could continue to do business even when their connection was down. And believe me, these connections went down a lot. 

But we've made a few mistakes when building this system. One of the biggest mistakes is that we stored account balances in the database and not just individual transactions. Now, it may seem like an optimization to store the card balances. After all, you want to check the card balance very quickly. We retained a complete history of all the transactions so we weren't loosing information by storing balances, but it turned out that this one design decision ended up being the single biggest source of problems with reliability, scalability, and correctness of the system. Why was this such a big mistake?

 ```
 SELECT Balance WHERE Card = X
 UPDATE SET Balance = Y WHERE Card = X
 ```
 
 By updating the card balance immediately upon receiving the transaction and not returning until it was updated, we were working hard to guarantee consistency because the next check for the current balance would have to include the transaction that we just received. What we didn't realize was that in order to guarantee consistency we had to forfeit availability. We didn't see this in the test environment because network partitions were pretty rare, but under load they occurred much more frequently.
 
 
 ##### CAP Theorem
 The CAP Theorem describes the behavior of a distributed system. This theorem talks about how the system reacts when it gets a write request followed by a read request. The theorem states that for any given pair of requests, a write followed by a read, a distributed system can promise to guarantee only two out of three attributes. These attributes are: 
 - Consistency
 - Availability
 - Partition tolerance
 **You can get any two at any given time, but you cannot have all three.You have to give up the third one for that particular pair of requests.**   
  - Consistency -the system guarantees to read data that is at least as fresh as what you just wrote. So, whether the client reads from the same node that I just wrote to or from a different node, that node is not allowed to return stale data. So, somebody else might have written something newer and the client might see their change, but consistency guarantees that the client will not see older data than what it just wrote. 
 - Availability means that a non-failing node will give the client a reasonable response within a reasonable amount of time. Now, all that's relative, but what that really means is that it won't hang indefinitely, and it won't return an error. This applies to both the read and to the write request. So, that means that the write request will acknowledge that the data was actually written, and the read request will return valid data. Neither of these requests can return an error, and neither one is allowed to hang indefinitely. So, this guarantee only applies to non-failing nodes. A node itself could actually be down, and the system would remain available. If the client is able to get access to any non-failing node and that node responds without an error in a reasonable amount of time, then the availability guarantee is upheld. 
 -Partition tolerance -  guarantees that a distributed system will continue to function in the face of network partitions. A network partition is a breaking connectivity. It means that nodes within the system cannot communicate with one another. A partition could be isolated to just the connection between two specific nodes or it could run through the entire network. On the other hand, a partition could be just a temporary loss of connectivity like maybe the loss of a single packet due to line noise or a partition could refer to something permanent like a backhoe cutting through a buried cable. But if the distributed system continues to function when the network is partitioned, then it's said to be partition tolerant.
 
 **The CAP Theorem says that we can only have two at any given point of the system. For any given pair of requests, we have to give up one of them.**  But shown in the Choices video, there are really your only two choices, forfeit consistency for availability or forfeit availability for consistency. You cannot consciously choose to give up partition tolerance because it's a fallacy to believe that the network is reliable. Network partitions are going to happen, and when they do your software will either be inconsistent or unavailable.
 
 **Proof explained @ 4:40**
#### Fallacies of Distributed Computing
The Fallacies of Distributed Computing ooints out the  differences between object-oriented programming that we're used to and network programming. They show us where bringing object-oriented assumptions into distributed systems can lead us astray. 
The Fallacies of Distributed Computing are: 
1. The network is reliable
2. Latency is zero
3. Bandwidth is infinite
4. The network is secure
5. Topology doesn't change
6.There is one administrator
7. Transport cost is zero
8. The network is homogeneous. 

**The Network is Reliable.** Many of the technologies that we'll be looking at will hide the fact that you're using a network at all to send requests to different machines. For example, if you've every used WCF then you've seen how it can quickly turn a web service call into a method call on an object. Method calls are completely reliable. When you make one call, you know for certain that the object that you're calling will receive its parameters and that it will execute the method. And when that method call returns, you know beyond a shadow of a doubt that the caller will receive the return value, and then the caller will continue executing at the next line. If the method throws an exception, then you can be absolutely sure that the caller will catch the exception and will take the appropriate action. But the same cannot be said about web service calls or indeed any communication over a network. When you make a web service call there's no guarantee that the service will receive the request, and when it returns a value you can't be sure that the caller will actually get the result. And when an exception is thrown, it can sometimes be hard to tell whether it was the result of a problem in the web service itself or simply some kind of a network failure. So, when coding for network communication, we have to understand that the call might fail in ways that simply are not possible in regular object-oriented in-memory programming. We have to code for failures at every point in the communication. The request may fail on its way to the server or worse yet the response may fail on its way back to the client. If a failure happens, then the client generally can't tell which side failed. They can't tell whether the message failed on its way to the service or the response back to the client, so that leaves the system in an indeterminate state. These kinds of states don't occur in regular object- oriented coding, so we typically don't think about them, but when the network is unreliable then we have to. We're going to spend the majority of this course battling this one fallacy. You can see how it's related to the CAP Theorem especially with regard to partition tolerance. We'll talk a little bit more on that later, but let's continue down the list of fallacies. 

**Latency is Zero.** So, requests take time. In most object-oriented code time isn't an issue. The code just tends to be procedural just moving from one statement to the next as quickly as the machine will allow, so we don't even thing about the dimension of time. But when we're sending our requests to a remote machine, we have to be aware of the passage of time. Until that response comes back, we're in a state of limbo where we don't know whether the request got through or not, and we have to timeouts for these requests. If the timeout elapses, then we don't know whether the request would have actually succeed had we just waited a little bit longer. We have two different machines each keeping track of its own state separated by time and space, and somehow we need to choreograph a dance between these two machines even though neither one really knows the state of the other. A lot of systems that I've build work fine on localhost, but then they don't work when deployed to real networks.

**Bandwidth is Infinite** When you pass a parameter to a method, the entire parameter makes it to the other side. Even if the parameter is a long list or a big XML document, the parameter that's actually passed is really just a pointer to some shared memory, and the method can access that shared memory just as easily as the caller could. But in a distributed system, there is no shared memory, so we just have to send the contents of that long list or that big XML document, and a common mistake that developers will make is to simply copy the whole thing into a single request. Unfortunately, the bigger the request is the more likely that it is to fail. In a distributed system, messages need to be broken down into chunks, but you have to be aware when you're doing this that each chunk is going to be individually subject to latency, and so these latencies are going to add up. So, it's kind of a balancing act between latency and bandwidth, and we have to choose the right balance in order to make our systems the most reliable. 

**The Network is Secure.** . There are many more attack vectors available to a hacker when a message is on the wire. 

**Topology Doesn't Change** (final module) Topology includes things like the platforms and the tools upon which your code is built, as well as which machines have access to which other machines. But rather than build those assumptions into the code, it's much better to construct the system in pieces that can be reconfigured to match the topology into which it's deployed.

**There is One Administrator.** (final module) When you're building a distributed application, the developer, you, are the administrator, but once it's out into production then that's going to be somebody else. In fact, it probably won't be just one person. You will probably have to give up the reins to somebody that runs the network, somebody else that runs the machines, and then somebody else that runs the applications. And furthermore, with cloud hosting becoming more prevalent, the administrator of the box is probably not even aware of what the applications are that are running on it, so you have to build your applications so that each different type of administrator can manage just the part of the system that they need.

**Transport Cost is Zero.** in object- oriented coding creating a new object is free. All it costs is a little bit of memory, and memory is cheap, but in a distributed system sending a message costs real money. The cost of keeping the network running so that's paying for people, electricity, and space, but then there's also the cost per message, so that's the actual bandwidth cost. It can be easy to forget while you're working in a test environment that each of these messages that you're sending is going to translate into real dollars. 

**The Network is Homogeneous** (final module) There are actually two fallacies in this one statement. 
- The first is about the stacks that the applications are written on. So, if all of the components are written on one stack, then we've got a homogeneous network. And this may be under your control for a small distributed system, but larger ones, especially those that integrate across company boundaries, are going to be written by different teams, and so you have to write each component with the knowledge that you're going to be using different vendors for different pieces of the system. But that's not the worst of the assumptions. 
- The second one is that all of the components will be on the same version. When we roll out a distributed system, this is probably going to be true, and this is going to lure us into a false sense of security, but when we reach version 2, we'll find ourselves, if only temporarily, in a heterogeneous network. We'll still have version 1 components running, but then we'll also want to deploy version 2 on top of them. Now, we might solve this problem in the beginning by just simply taking down the whole system, upgrading it all, and then bringing it back up. That's only going to work for a short period, and maybe not even ever. As more and more versions are deployed and more and more components are added, the window of time during which we can have multiple versions is going to grow. And once we have two or more project teams or two or more companies involved, then that window is basically going to stay open forever. We will never again have all the components on the same version, and it will be impossible to bring down the whole system to do an upgrade. So, we have to build our systems with the assumption that we're going to have a heterogeneous system. We're going to have some older versions of components interoperating with newer versions of components, and we have to build our systems to be resilient and tolerant of this. The most dangerous part of this fallacy is the fact that our one and only chance to combat it has already come and gone before we've deployed the first version. Once we have version 1 in production, then we have already sealed our fate. If we didn't build the system with the ability to be heterogeneous from the outset, then it's very rare that we'll ever get it back, so we're going to need to spend some time in module 7 making sure that we build this capability of creating a heterogeneous system from the start. 

#### Domain Driven Design
Let the problem domain drive the software design and not try to make the problem conform to the capabilities of the software. Too often we approach a problem by modeling it as rows and columns in a database and then writing CRUD operations to manage that data. When we deliver such applications to the business owners, they can often find no relationship between the data grids and input forms that we've delivered and the actual business processes and workflows that they were trying to model. Domain Driven Design seeks to change all that. **Creating N-Tier Applications in C#. Also, check your Pluralsight feed for a future course on DDD**

The three core concepts are: (Example: PharmaNet)
### Ubiquitous Language 
The process of Domain Driven Design begins by agreeing upon a ubiquitous language between the developers and the business owners. This isn't something that we can just jot down in one or two meetings. This is a continuous cultivation and curation of language over the entire lifetime of the project. All of the code is modeled and written using the terms from this ubiquitous language, so any change to the ubiquitous language necessitates a change to the model. That indicates that the understanding has evolved in some way, and so the model has to evolve to capture that new understanding. A ubiquitous language allows developers and business owners to communicate with one another without having to translate.

### Bounded Context
 In the PharmaNet example,  we found that we had three bounded contexts:
 - rebates 
 - sales
 - performance
 
 The rebates bounded context captured all the rules of the rebate contracts. It was primarily the domain of the account manager. They would specify the measured product groups for each of the rebates, their measurement methods, and the awarded tiers, and they would set up the roles of the contracts. 
 
 **The Rebate system is optimized for providing the account manager with the capability of editing the roles of a rebate contract but Once a rebate enters the performance context it needs to be optimized for a different purpose.**
 
 **Also, in the rebate context, the data structures used tend to be more mutable and interactive so that the account managers could make changes**
 
 The sales bounded context recorded the sales history of each member. Information from the e-commerce and fulfillment systems fed into the sales area.
 
 The performance bounded context calculated the performance of each member toward each of the rebates. It combined the rebated specifications with the sales data to perform its calculations. It presented the outcome of these calculations in the form of report cards for the members to view online. We found this breakdown of the problem into bounded context to be preferable to modeling the entire domain as one enterprise data model. We found that for each context we were trying to solve a specific problem. We could optimize that model specifically for that problem and not have to compromise in order to allow some other context to share the model.
  
 **On the other hand, the performance context needs to be optimized for rapid calculation. The data structures that we chose in each context were subtly different to satisfy each need.**
 
 **Also, in the performance context, the data structures tended toward immutable and pluggable strategies so that we could form them into a pipeline of calculations.**
 
 Bounded contexts also offer benefits over enterprise data models in terms of the CAP Theorem. An enterprise data model represents one tightly interconnected cluster of nodes in the distributed system where consistency is prioritized over availability. Any party that needs a consistent view of any part of the enterprise needs to make a connection to this one cluster. The more consumers that connect to it, the less available it becomes. The only way to combat this trend is to reduce the likelihood of a network partition by spending more money on expensive hardware and network management. **An enterprise data model forces you to scale up rather than scaling out, but a bounded context represents a smaller cluster of nodes.** It's decoupled from the other contexts. Because it has its own model, it's not reliant upon its connections to those other contexts. It can make decisions based only on the information that it has on hand. A bounded context could be written to favor availability over consistency. If the system of record is down, then the downstream system is unaffected. It already has a cache of the data that it needs organized and optimized for the problems that it is designed to solve. This isn't automatic. You have to conscientiously choose to build this kind of capability using CQRS, Event Sourcing,etc. 

### Aggregate Roots
An aggregate root is the top-level entity of an aggregate of children and grandchildren. Business operations are performed on the aggregate as a whole and not on the individual objects within it. The root is the only entity within the aggregate that has global identity, and other aggregates cannot reference any of its children directly. And the root controls how an aggregate is queried, altered, and deleted.

### CQRS
CQRS stands for Command Query Responsibility Segregation. The pattern is based on a guiding principle called Command Query Separation, but the pattern is applicable to very specific circumstances. 
### Command Query Separation

Command Query Separation states that a method should either change the state of an object or return a result, but not both. When we're following this principle, we'll separate our methods into these two sets:

- Commands- those that can change state and a command will return void.
- Query - those that return results. A query will declare a return type. 

 When you separate the methods like this, the caller can be confident that they can execute whatever sequence of queries they need to in order to arrive at a desired result. Then they can make a decision based on that result and record that decision with a single command. This style of use is much easier to reason about because there's only one state change, and that occurs at the end of the sequence. 
 
###  Command Query Responsibility Segregation
Command Query Responsibility Segregation on the other hand exists to solve a specific problem. It's not a guiding principle, and it's not broadly applicable.

### Collaborative Domains
The problem that Command Query Responsibility Segregation is really good at solving is a problem of blocking the user when locking the data. 

Example: We have a lot of people who want to reserve seats on an airplane. Some prefer window seats. Others prefer aisles. Some are traveling with children and need to be seated together. There are probably enough seats to satisfy everybody's needs, but there's still a problem. If everyone picks the one seat or the set of seats that they want then that's going to lead to some strange behavior. Invariably, a few people are going to be able to reserve the seat that they want, but then other people are going to be competing for the same seats. The people who don't get the seat that they want are going to have to try again, and they might have to try several times before they can finally find a seat that will suit them. But this problem gets even worse for the family that's trying to get seats together. They might get one or two seats, but then lose the third. They have to give up the seats that they've already reserved in order to find a set that's all together, so they end up playing this game of musical chairs that's not really much fun for anybody. 

This kind of problem is called a collaborative domain. You have a large set of people collaborating on a small set of data. Whenever this occurs you run the risk of forcing the users to try and then retry their operations until they get the desired result. Locking the data is necessary, that's how you ensure that only one person gets a seat, but blocking the user while you do that is not. 

Let's see how else we could solve this problem without blocking the user. Let's introduce a ticketing agent. All of the passengers will give her the specifications for the seats that they want. Some will say I'd like an aisle seat, others will say that I'd like three seats together, and then this ticketing agent gathers all of these requests and then assigns the seats. If she can't assign a passenger the seat that they want then she just says I'm sorry but I reserved a different seat for you. Would you like to keep this reservation or cancel it? This gets the passenger out of the chaos of trying and retrying the seat assignments, and instead it gives that responsibility to just one person. After everything settles, the passengers can query the airplane to see which seats they've been assigned, and they can also see which seats were assigned to other passengers and which seats remain vacant, but what they can't do is change the seat assignments. They can't simply talk to this airplane model and say no I don't want that one. I want a different seat. For that they'll have to go back to the ticketing agent. This s a pretty big tradeoff in the way that we interact with systems. Most of the time we want immediate feedback from a program showing us the change that we have just made. We want an interactive experience from the system. So, for the most part we really don't want to apply CQRS in a lot of situations. The only place where it makes sense is in these collaborative domains whenever we have a large number of people all working together on a small set of data. That's when CQRS can kind of bring order to the chaos. 

###  Pharmanet Example : 
The fulfillment system feeds into sales and has a large number of customers, and they're all placing orders, and all those orders have to be picked from a very small number of warehouses. And so we've got a large number of users all competing for the same small set of resources on the same small set of data, and so this is actually a perfect example of a collaborative domain. So, let's go ahead and take a closer look at the fulfillment bounded context.


