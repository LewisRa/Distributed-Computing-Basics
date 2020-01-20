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

