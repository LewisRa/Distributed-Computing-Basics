
# Patterns for Building Distributed Systems for The Enterprise
### Patterns
- CQRS
- Event Sourcing
- Domain Driven Design
- Event Driven Architectures

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
 - Network partition tolerance <br>
 **You can get any two at any given time, but you cannot have all three.You have to give up the third one for that particular pair of requests.**   
  - **Consistency** -the system guarantees to read data that is at least as fresh as what you just wrote. So, whether the client reads from the same node that I just wrote to or from a different node, that node is not allowed to return stale data. So, somebody else might have written something newer and the client might see their change, but consistency guarantees that the client will not see older data than what it just wrote. 
 - **Availability** means that a non-failing node will give the client a reasonable response within a reasonable amount of time. Now, all that's relative, but what that really means is that it won't hang indefinitely, and it won't return an error. This applies to both the read and to the write request. So, that means that the write request will acknowledge that the data was actually written, and the read request will return valid data. Neither of these requests can return an error, and neither one is allowed to hang indefinitely. So, this guarantee only applies to non-failing nodes. A node itself could actually be down, and the system would remain available. If the client is able to get access to any non-failing node and that node responds without an error in a reasonable amount of time, then the availability guarantee is upheld. 
 -**Network Partition tolerance** -  guarantees that a distributed system will continue to function in the face of network partitions. A network partition is a breaking connectivity. It means that nodes within the system cannot communicate with one another. A partition could be isolated to just the connection between two specific nodes or it could run through the entire network. On the other hand, a partition could be just a temporary loss of connectivity like maybe the loss of a single packet due to line noise or a partition could refer to something permanent like a backhoe cutting through a buried cable. But if the distributed system continues to function when the network is partitioned, then it's said to be partition tolerant.
 
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

## Domain Driven Design
Let the problem domain drive the software design and not try to make the problem conform to the capabilities of the software. Too often we approach a problem by modeling it as rows and columns in a database and then writing CRUD operations to manage that data. When we deliver such applications to the business owners, they can often find no relationship between the data grids and input forms that we've delivered and the actual business processes and workflows that they were trying to model. Domain Driven Design seeks to change all that. **Creating N-Tier Applications in C#. Also, check your Pluralsight feed for a future course on DDD**

The three core concepts are: (Example: PharmaNet)
#### Ubiquitous Language 
The process of Domain Driven Design begins by agreeing upon a ubiquitous language between the developers and the business owners. This isn't something that we can just jot down in one or two meetings. This is a continuous cultivation and curation of language over the entire lifetime of the project. All of the code is modeled and written using the terms from this ubiquitous language, so any change to the ubiquitous language necessitates a change to the model. That indicates that the understanding has evolved in some way, and so the model has to evolve to capture that new understanding. A ubiquitous language allows developers and business owners to communicate with one another without having to translate.

#### Bounded Context
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

#### Aggregate Roots
An aggregate root is the top-level entity of an aggregate of children and grandchildren. Business operations are performed on the aggregate as a whole and not on the individual objects within it. The root is the only entity within the aggregate that has global identity, and other aggregates cannot reference any of its children directly. And the root controls how an aggregate is queried, altered, and deleted.

##
CQRS
CQRS stands for Command Query Responsibility Segregation. The pattern is based on a guiding principle called Command Query Separation, but the pattern is applicable to very specific circumstances. 
#### Command Query Separation

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

## Durability Guarantees
 So far in the PharmaNet application we've created a FulfillmentService that will take the user's order before it processes it. By separating the command PlaceOrder from the query CheckOrderStatus we've given the service a little more flexibility and allowed it to resolve some problems with efficiency and scalability. Unfortunately, we also introduced some problems. We're not guaranteeing the durability of messages.
 
 **Durability is the promise that we handle a message once and only once.** We can break durability down into three guarantees. Let's see what they are. We expose the PlaceOrder command as a WCF service. Currently, when the client calls PlaceOrder, we put their message into an in-memory queue, and then we return from that method. Returning from the method causes WCF to return an HTTP 200 response to the client. If the IIS application pool is recycled, then the in-memory queue disappears, and it takes with it all of the messages that haven't yet been processed. By returning that HTTP 200 we promised that we would take the order, but we aren't actually going to uphold that promise because the order is gone. What we need to do is receive the message and persist it. **The first durability guarantee is the message is persisted.**
 
 The next problem that could occur happens when we take the message off the queue. If a failure occurs when we try to process the message, for example if the database connection times out, then the message is lost. It was already removed from the queue, so it will never get processed. That brings us to our second guarantee. **The message is not removed from the persistent storage until it is processed.**
 
 There's one more thing that could go wrong. When the customer sends us an order by calling PlaceOrder, we're going to store it in the persistent queue, but then suppose that the HTTP 200 message is lost. Remember the fallacies of distributed computing. The network is not reliable. We cannot guarantee that the HTTP 200 has gotten to the client, so in that case the client is obligated to retry. It will send us another copy of that order. That will result in a duplicate message in the persistent queue. We're going to process the first message, but then we need to recognize that the second one represents the same order and ignore it. **The third guarantee is that duplicate messages are recognized.** If we don't recognize duplicate messages, we'll end up corrupting the data. 
 
 #### MSMQ Options
There are several mechanisms that we could use to persist the incoming messages so that we don't lose them before we process them. There's not enough time in this course to go through all of those options, so we'll just focus on the one that's built into Windows, and that's **Microsoft Message Queueing or MSMQ. MSMQ has the advantage of being able to participate in transactions with SQL Server, and the thing that bridges that gap is the Distributed Transaction Coordinator or MSDTC.** 

**Messages put into MSMQ can be either:**
- recoverable or non-recoverable where a recoverable message is persisted to disk and A non-recoverable message is not. (Non-recoverable messages have a significant speed advantage over recoverable messages)

**A queue can either be:**
- transactional or non-transactional. A transactional queue can participate in transactions with SQL Server using MSDTC. A non-transactional queue cannot. The Distributed Transaction Coordinator imposes some additional overhead and has been known to be somewhat temperamental at times, so it should be used sparingly, but in this case this is exactly the behavior that we need. We need to ensure that the messages removed from the queue and the transaction is completed in SQL Server in an atomic fashion, and so we will choose transactional queues. 
- A queue can either be public or private. Public queues are registered with Active Directory so that they can be discovered anywhere in the enterprise, but private queues are registered with only the machine that they reside on. So, to address a private queue, you have to know the address of the machine. For our implementation of the CQRS pattern, we're going to be using a queue as an inbox on a specific machine so the machine is known and we don't need to take a dependency on Active Directory, and so we will choose private queues. 

For PhamaNet, we use recoverable messages stored in transactional queues that are private to one machine. 

#### Enable MSMQ and MSDTC
To enable MSMQ:  go to the Control Panel, go in to Programs, and then I can turn on Windows Features. And then I open up Microsoft Message Queuing. The minimum that you'll want to check is MSMQ HTTP Support, which will also bring in the core services. You can turn on these other options if you choose, but for most purposes HTTP Support is going to be sufficient. Then we want to turn on the Distributed Transaction Coordinator, so for that I'm going to run dcomcnfg. That's D-C-O-M-C-N-F-G. When I run that, that's going to bring up my Component Services. I can open that up, go to My Computer, and when I right-click and go to Properties I've got an MSDTC tab that's the Microsoft Distributed Transaction Coordinator, and I want to be sure that use local coordinator is checked. And then with that checked, I can go into the Distributed Transaction Coordinator Folder, right-click on Local DTC, and go to Properties. And for this demo, the default Security Settings with everything turned off are going to be sufficient because I'll be using the Distributed Transaction Coordinator with a private queue and SQL Server running on this box. But if I were to access SQL Server on a different box, then I would want to turn on the Security Settings. So, then I would want to enable the Network DTC Access, Allow the Remote Clients, but I don't need to Allow Remote Administration, Allow Inbound and Outbound, and for simplicity sake because I'm not relying upon Active Directory for anything I'll use No Authentication. Then I'll want to enable the XA Transactions so that I can share those transactions with SQL Server, and this will be the configuration that I use when I have SQL Server running on a different machine. And on that machine I will also need to configure the Security Settings for MSDTC. But since I'm doing everything locally, I don't need to enable any of those settings in order for this next part to work. So, now let's go into our code and change things up so that we're using MSMQ and the Distributed Transaction Coordinator from our code.



when we're trying to move large
quantities of data often the tool of
choice is an ETL tool which stands for
extract transform and load
however when we're communicating between
individual application processes we
often use an enterprise service bus or
ESB 

 unique challenges
of moving bulk data vs as real-time
transaction data 

on
the other hand requires a real-time
service to be constantly monitoring
transactions this is because ESB tools
are often talking between applications
for example imagine you have a closed
opportunity in your crm and you want
that closed opportunity to trigger an
invoicing process for your accounts
receivable system now rather than hand
entering all of the information you
would set up a service bus which would
carry the relevant information to your
accounts receivable group speeding up
the invoice creation process now this
would be akin to using a bicycle in a
previous example it's not great for
moving

etl and esb is middleware
latency flexibility
and transformation flexibility

we're going to understand how this tool works. There are two basic kinds of Enterprise Service Bus. There's the brokered service bus and the decentralized service bus. A brokered service bus is characterized by having a single point of configuration. This is one place where you can go to deploy your services and configure them. For that reason, this type of service bus tends to support a single governing body that is very hands on in the enterprise. This might be a change management team, or an operations team, or maybe even a system architecture team. That one governing body deploys the services and configures them. The advantage of using a brokered service bus is that the middleware is capable of making decisions on message routing and prioritization. Typical brokered service busses are products like TIBCO, BizTalk, and Web Sphere. The second kind of enterprise service bus is decentralized. These tend to be collections of independently configurable services. They're characterized by a collaboration among many independent teams. Whereas a brokered service bus has intelligent middleware for making routing and prioritization decisions, a decentralized service bus relies upon the services themselves to make all these decisions. The middleware is pretty dumb. Some typical decentralized services busses are NServiceBus, MassTransit, and Rhino Service Bus

use the service bus to scale our queries independently of our commands. The next thing that we build is an integration between bounded contexts. We're going to build this integration by publishing events. And then third, we're going to manage dependencies between those bounded contexts. We will do this by subscribing to events.
