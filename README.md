# Distributed-Computing-Basics

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

### Asynchronous
EXAMPLE: **Amazon.com**

What happens when you click the **Confirm Purchase** button on Amazon?
- What do you see?
Your computer gets redirected to a succes screen that: 
- "Thanks for you purchase, please shop with us again!"

