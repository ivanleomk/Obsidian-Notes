# Questions

Here are some questions I came up with 

1. How many pages to crawl per hour / month?
2. How long should we be storing data for?
3. Will we be crawling any protected pages?
4. How do we determine what to crawl?

Here are some recommended questions

1. What is the main purpose of the search index crawler
2. How many web pages does the crawler collect per month?
3. What content types are included?
4. How should we handle duplicate pages?

# Estimation

Figures given ~ Need to store for around 5 years and the crawler will hit around 1 billion pages per month

Estimates
- QPS : 1 billion / 12 / 30/ 24 / 3600 = ~ 400 pages/second
- Peak QPS : ~ 800 pages/ second
- Assume that each page is ~ 500kb, this means that per month we need about 500 TB of space per month.
- Assume that data is stored for 5 years, this means that we need ~ 30 PB of storage to store five years of content

# Architecture

![[Pasted image 20230905130136.png | 500]]


1. **Seed URL** : Initial starting point of URLs
2. **URL Frontier** : List of URLs to be crawled ( page content has not been downloaded yet )
3. **DNS Resolver** : Given a URL, we get the IP Addr 
4. **HTML Downloader** : Download the HTML content of the page given a url
5. **Content Parser** : Parse HTML content and determine if page has been crawled correctly or not
6. **Content Seen** : Compute a MD5 Hash of the page content and determine if we've seen it before - possible to use a hash table or a bloom filter for this
7. **Content Storage** : We use a combination of disc storage and RAM in order to store our content. Recently accessed/popular sites might be kept in storage while others are not. 
8. **Link Extractor** : Extract out all of the different links in the extracted HTML and convert to an absolute url link
9. **URL Filter** : Remove any undesirable links - eg. those that are blacklisted or belong to sites which should not be crawled
10. **URL Seen** : New Urls that were extracted in the link extractor are then passed to a single component whose's sole purpose is to identify whether or not to add it to the URL Frontier or not

# Considerations

Ideally, we want to make sure that we crawl sites in a friendly manner. A good way to do so is to combine a BFS approach with some additional optimisations.

![[Pasted image 20230905131657.png | 400]]

- **Politeness** : We do not want to overwhelm servers with our requests. As a result, we only allow worker threads to crawl a specific subset of assigned URLs. This is done by maintaining a map of url <> Queue. URLs are added to a specific queue and worker threads consume new URLs that are added to that queue. In short, our queues to worker threads share a one-to-many relationship.

- **Prioritisation** : We need to make sure that we crawl websites according to their relevance. Therefore we utilise a prioritiser service which is able to decide on the prioritisation of a url. It then adds it to a specific queue that corresponds to its assigned priority.  
  
- **Memory Speed**  : We maintain a buffer in memory to allow for us to read/write quickly to the disc space.

- **Timeout** : We implement a timeout in order to make sure that we don't spend too much time waiting on a host. If a host does not respond within a predefined time, we will move on to another URL
  
- **Consistent Hashing**  : We can add/remove server processes using consistent hashing. This will help us to prevent system errors in the event that specific worker processes go down.
  
-  **Server-Side Rendering** : We can utilise a javascript process to render pages that are using scripts to generate links/content on the fly

