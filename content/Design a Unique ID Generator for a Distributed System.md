> A unique id generator helps us to be able to work on

# Questions

Some useful questions to keep in mind are

1. What form should the IDs take? 
2. Is there a strict numerical sequence for these IDs to follow - for each new record does the ID increment by 1?
3. Do IDs only contain numerical values?
4. What is the ID length requirement?
5. What is the scale of the system?

# Strategies

## Multi Master Replication

![[Pasted image 20230829230921.png]]
Instead of incrementing each new row by 1, we increment it by $k$, where $k$ represents the number of database servers that are in use. 

However, this does not scale well given that in distributed systems, servers are added and removed quite rapidly all the time

## UUID Generator

We simply use a UUID number - this is a 128 bit number which has a very low probability of having a collision.

> After generating 1 billion UUIDs every second for approximately 100 years would the probability of creating a single duplicate reach 50%

In fact, this is simple because each web server can operate independently of each other. But, there are three main issues here

- Size of UUID is 128 bit
- IDs cannot be sorted by the time they were created ( in the event that we need a system whereby IDs generated in the evening must be larger than those generated in the morning )
- IDs can be non numeric

## Ticket Server

Ticket Servers utilise a centralised database server and multiple consumers.

![[Pasted image 20230829232056.png | 500]]

However, using a single Ticket server introduces the risk of having a single point of failure. As such, all systems that depend on it will face issues.

## Twitter Snowflake

Twitter utilises a unique system called **Snowflake** in order to generate its requirements. Instead of generating the ID Directly, we divide the ID into different sections

![[Pasted image 20230829232328.png | 500]]

- Sign bit: 1 bit. It will always be 0. This is reserved for future uses. It can potentially be used to distinguish between signed and unsigned numbers.
- Timestamp: 41 bits. Milliseconds since the epoch or custom epoch. We use Twitter snowflake default epoch 1288834974657, equivalent to Nov 04, 2010, 01:42:54 UTC.
- Datacenter ID: 5 bits, which gives us 2 ^ 5 = 32 datacenters.
- Machine ID: 5 bits, which gives us 2 ^ 5 = 32 machines per datacenter.
- Sequence number: 12 bits. For every ID generated on that machine/process, the sequence number is incremented by 1. The number is reset to 0 every millisecond.

# Design

**tl;dr** We utilise twitter's snowflake system. 

> The most important 41 bits make up the timestamp section. As timestamps grow with time, IDs are sortable by time. Figure 7 shows an example of how binary representation is converted to UTC. You can also convert UTC back to binary representation using a similar method.