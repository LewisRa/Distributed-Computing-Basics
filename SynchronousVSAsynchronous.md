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

You decide to integrate with Salesforce. You want to keep a copy of important Salesforce data onyour system to track some analytics with you data.

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
