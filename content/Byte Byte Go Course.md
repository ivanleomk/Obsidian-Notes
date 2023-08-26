# Components of An Application

![[Pasted image 20230824164705.png | 500]]

## DNS

> A DNS is used to resolve a website address (Eg. www.google.com ) to a concrete IP Address

Our browser will hit a browser in order to get back an IP to forward an address to. Typically a DNS is a service provided by another service (Eg. Cloudflare )

## Servers

Our application then hit the servers and get back either HTML or JSON content and renders it on the screen

### Scaling

When we scale our servers, we have two choices - [[Vertical Scaling]] or [[Horizontal Scaling]]. Vertical Scaling involves increases the hardware that our server is hosted upon while horizontal scaling involves increasing the number of instances of the server. 

### Statefulness

Our servers are stateful if they hold access to some unique form of information on the servers instead of storing it in some stateful store. The following is an example of a stateless architecture where user sessions are stored in a common data store rather than on the server process itself.

![[Pasted image 20230824173646.png | 500]]

## Database

We have two forms of databases

1. [[Relational Databases]] : These are used to store and keep values in tables and rows. They provide an easy way to join data.
2. [[Non Relational Databases]] : These are newer and tend to include things like NoSQL 

We ideally want to use Non Relational Databases when we're dealing with applications that require low latency, unstructured or massive amounts of data. 

### Master/Slave

When we reach a high enough volume, we'd ideally want to migrate over to a master-slave configuration. 

![[Pasted image 20230824170610.png | 500]]

Possible Events

- **Master DB going down** : In the event that our master DB goes offline, then a slave database is prompted to temporarily become the master DB
  
- **Slave DB Unavailable** : Master DB takes over the role of a Slave instance temporarily while we await the provisioning of a new Slave DB.
### Scaling

When we want to scale our databases, we have two options - [[Horizontal Scaling]] or [[Vertical Scaling]]. Vertically scaling our database involves increasing its overall capacity while horizontally scaling our database involves increasing the number of instances we have in the network.

Note that horizontally scaling our database will result in a larger amount of issues with data consistency. We might also have to denormalize our data to support the new sharded data in our databases

## Cache

When we're working with data, sometimes, we might want to add a cache. This is an intermediate layer between our database and the server.

![[Pasted image 20230824172147.png]]

Typically, we want to use a cache for data that is read frequently but accessed infrequently. When defining a cache, we need two things

- **Eviction Policy** : How do we remove items when we're reaching 
- **Expiration Policy** : How long do we preserve data for? 
- **Failure Mitigation** : A single cache server represents a SPOF. How can we scale out the number of instances while preserving the cache 
- **Consistency** : This involves keeping the data store and the cache in sync. 

### Applications

#### CDN

A frequent use of a cache is a CDN, otherwise known as a Content Delivery Network. It is a collection of servers which are ideally located geographically close to your users. This reduces latency for users since the request can be resolved in a much shorter duration of time.

![[Pasted image 20230824173223.png]]

Typically we'd implement the following to ensure our cache works well

- **Object Versioning** : This helps to ensure we are always serving up the correct version of the model.
- **CDN Fallback** : In the event that our CDN is unreachable, we need to implement a manual failover to get around it
- **Invalidation Lifecycle** : We need to make sure that we use APIs provided by CDN vendors to properly ensure validity of objects


## Queues

A message queue is a durable component, stored in memory, that supports asynchronous communication. It serves as a buffer and distributes asynchronous requests.

![[Pasted image 20230824173846.png]]

Producers create jobs published as Messages and Consumers consume these jobs.

Decoupling makes the message queue a preferred architecture for building a scalable and reliable application. With the message queue, the producer can post a message to the queue when the consumer is unavailable to process it. The consumer can read messages from the queue even when the producer is unavailable.
# Interviewing

System Design interviews will typically be ~ 45 minutes to an hour. It's important to note that **you are being evaluated on how well you work collaboratively and your ability to resolve ambiguity**.

Here is a rough breakdown of the components
- Understanding Requirements ( 3-10 minutes )
- Creating a High Level Design ( 10-15 minutes )
- Deep Dive into specific components ( 10-25 minutes )
- Conclusion ( 3 - 5 minutes )

## Understanding the Requirements

There are no marks for replying fast - in fact you'll be penalised for giving a quick response with a clear understanding of the question. Therefore, take time to understand the specific requirements of the question/

Here are some useful questions
- How is the application required to scale?
- What are the main features that we need to implement?
- What is the rough load that the application will need to handle?
- Are there any specific latency requirements?

## Creating a High Level Design

It's important here to **get the interviewer's buy in to your solution**. At this point, you should have 

1. Obtained a clear understanding of the features that you need to support
2. Identified any potential bottlenecks that your system might be forced to support.
3. Done some basic estimation as to whether your scale constraints can be matched by your proposed design
4. Have a clear implementation of a architecture that supports the features that you need to support

## Deep Dive

At this stage, depending on your experience level and the interviewer, you might need to focus on different things. So, look for hints to see what your interviewer might like.

## Wrap Up

Some good things to do here are

- Give a short recap on what you've done
- Talk about specific constraints that your system has - Eg. it cannot support Y feature at the moment and might have issues down the line with Z
- Talk about how to scale up - how can we improve its ability to do more
- Error Logging and Metrics to track - How can we verify that the system is indeed working as intended.



