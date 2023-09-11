---
title: Design Youtube
date: 2023-09-11
---
> Youtube is a streaming video site. People upload videos and others watch them :)



# Questions

I generated the following questions

- How many videos per second upload?
- How many users will be watching?
- What storage requirements do we need to consider?

Some other good questions would be

- What features are important?
- What clients to support?
- How many daily users are we expecting?
- Do we need to perform encryption on the videos uploaded?
- Is there a file size requirement? What is the maximum file size that we need to support
- Can we utilise cloud infrastructure products?

## Calculation

- Assuming that we have 5 million daily active users (DAU)
	- Each user watches 5 videos per day
	- Each video is approximately 300mb
	-  Approximately 10% of users upload a new video a day
	- It costs us approximately $0.02 / gb to serve these videos 
- Then we have 
	- 5 million * 0.3 * 5 * 0.02 = 150,000/day of CDN costs
	- 0.1 * 5 million * 300 MB = 0.03 * 5 million GB = 150 TB


# Design

Our application has two main features to support

1. Video Uploading
2. Video Streaming

## Video Uploading

![[Pasted image 20230911123142.png | 400]]

Video uploading should take the following form

1. **Binary Blob Upload** : First, upload a binary blob of the raw video onto our storage. At the same time, we also update metadata **in parallel** so that users immediately see information on their uploaded file - this includes data such as file name, format, size etc.
2. **Transcoding** : Once we've successfully uploaded the binary blob, then we trigger a transcoding job. 
3. **Completion Handler** : Once we've processed the video, we need to update our database with the relevant metadata and information about the video that we've processed
4. **CDN Propagation** : When we have the transcoded video, we can then propagate it to a CDN of our choice so that users can have quick access to it. 

### Upload

Some quick optimisations for upload include

1. **Allocating Buckets by Geographical Location** : We can allocate users buckets that are closer to their geographical location, thus speeding up the upload process
2. **Pre-Signed URLs** : This exposes the endpoint for uploads for a set duration of time and makes sure that only authenticated users can upload videos
3. **Protect Videos** : We can ensure that we protect videos by using a [[DRM System]] , [[AES Encryption]] or Visual Watermarking

### Transcoding

When we transcode a video, we are encoding it into another bitrate (Eg. it was uploaded in 1080p, and we generate a 360p and 480p version). Transcoding is useful because it allows us to reduce the quality of the video when networks conditions change for the worst.

We can visualise the transcoding process using a [[Directed Acyclic Graph]] model. This is similar to what is done in libraries like [[Airflow]]. 

![[Pasted image 20230911124057.png | 500]]

We can further optimise this by splitting up the video using [[GOP Alignment]] which allows us to further parallelise the operations. This can be done on the client side or the server side ( in the event that the client does not support the video encoding/type )

### DAG Scheduler

![[Pasted image 20230911124505.png]]

We can utilise the following architecture to support the distributed queue of tasks that we need to process for each worker. 

- **Preprocessor** : This handles the splitting of the video stream into smaller chunks using [[GOP Alignment]].
- **DAG Scheduler** : We then utilise a program which will split the DAG graph into stages of tasks and put them into task queues into the resource manager. Since the DAG specifies a clear dependency for each task, we know that tasks will be executed in the correct order.
- **Resource Manager** : This helps to manage the efficiency of resource allocation
- **Task Workers** : These workers run the tasks that are defined in the DAG. Different workers run different tasks
- **Temporary Storage** : This is used to cache metadata, or any other intermediate data that is used by workers in between jobs



## Streaming

![[Pasted image 20230911125333.png | 400]]


When users are watching the content, we want to make sure that they can start watching it as soon as possible. Therefore, we stream the video in chunks to their device.

In our case, since we've uploaded our videos to the CDN layer, we can stream directly from the edge server closest to the User. 


### CDN Cost Optimisation

As mentioned earlier, it becomes expensive to stream all content from the CDN since we have a large amount of data being processed. As a result, we can instead

- **Only use CDN for Popular Videos** : We limit the use of the CDN for most popular videos and stream other videos from high capacity storage video servers
- **Only Encode Popular Videos** : We can encode less popular/short videos on demand.
- **Location-Specific CDN Propagation** : We can only distribute videos on the CDN if they are popular in that specific region

# Concepts

-