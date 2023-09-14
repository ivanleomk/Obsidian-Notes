---
title: Design Proximity Service
date: 2023-09-14
---

# Questions

I came up with the questions of

- What are the important features?
- How many items to return?
- What criteria should we use to filter businesses
- What latency requirements?
- How many active users/queries are we expecting


These were some additional suggested questions
- What is the search radius that we are supporting?
- Can a user change the search radius on the UI?
- How does business information get added or deleted?
- Do we need to refresh the page as the user moves?

We can assume that with 
- Approximately 100 million daily active users (DAU) and about 200 million businesses and each user making about 5 queries a day
	- We have about 5 * 100 million / 10^5 ~ 5000 queries/second ( Assuming 10^5 seconds in a day for easy calculation )
	- This means that we have a peak qps of 10000 queries/second

# Design

![[Pasted image 20230914155206.png | 400]]

We want to provide 2 new APIs in our system

1. **GET** /v1/search/nearby : This returns a list of businesses based on certain search criteria. This consists of a paginated set of results to reduce server load.
2. Business APIs to help get and update information on businesses

Generally, since we have a significantly higher read to write ratio, we want to make sure that we can get data quickly. Updates can be applied on a nightly cron job.

There are two good choices to use in an interview

|               | Geohash                                                              | Quadtree |
| ------------- | -------------------------------------------------------------------- | -------- |
| Advantages    | Easy to use and supports radius search                               |      Supports K-nearest businesses by default, thus allowing for precision level to be modified    |
| Disadvantages | Difficult to adjust precision level since geohash precision is fixed |     Difficult to build the tree and complicated to rebuild the index     |

Geohash is much simpler to implement so we choose it for now. Since we can map each (lat,long) to a unique geohash, we can therefore store information in our database using a composite key of (geohash,business_id). This is better because addition and removal of a business are very simple tasks.

## Improving Throughput

In order to scale up our index, we can simply utilise more replicas. This is because the geospatial index is not large ( quadtree index will only take about 1.7 GB and we can assume that geohash index will be roughly equivalent ).

Sharding this database would mean that we need to add sharding logic into the application layer. But since everything can fit within the working set of the database server, then there is no need to shard the database.

We can also utilise the geohash of the user's coordinates as a cache key -> list of business IDs. So ideally, we have 2 different caches - one for business IDs and another for businesses.





# Concepts

## Geo Indexing

There are a few different ways to index our geographical information

1. **Two Dimensional Search** : Given a database with lat and long of various entries, we simply do `GET ALL ENTITIES WHERE dist(lat,long) FROM my center < specified value`. This is extremely inefficient since our dataset contains lots of data. Even if we built an index based on lat and long, we need to perform a large intersect operation - this is highly inefficient
   
2. **Evenly Divided Grid**: This is relatively simple to implement - we divide the world into small grids and then get all the entities which belong to a specific grid. However, problem is that the distribution is uneven ( Eg, we have deserts and New York City of the same size ) and so it is not ideal since we need to do a lot more filtering
   
3. **Geohash**: Take a latitude and longitude and recursively hash it. Each time we subdivide a grid, we add an additional bit to it. Geohash has the following specifications. Ideally, we fetch a geohash and all its surrounding neighbours. If we don't have enough businesses, then we simply remove the last digit of the geohash and then fetch all hashes which match the prefix (Eg. We originally have geohash of abcdef, we don't get enough so we get all that start with abcde )
   
   
![[CleanShot 2023-09-14 at 15.23.01@2x.png | 400]]

4. **Quadtree** : We build an in-memory database which continues to subdivide nodes until we reach some stopping condition (Eg. number of businesses). At each node, we store the top left coordinates and bottom right coordinates along with a pointer towards  its child nodes. 
   
   Our quadtree will take an estimated ~1.7 GB of storage. Note that updating a quadtree is significantly more complicated than using a geohash

![[Pasted image 20230914153207.png | 400]]


5. **S2** : Google S2 Geometry library is another big player which maps 2D points on a sphere to a 1D point. This makes it easy and more efficient to search for values.

