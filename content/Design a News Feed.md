> A news feed is a application view which displays a collection of media sorted according to some criteria

# Questions

I came up with the following questions

1. How many users do we have?
2. How many stories do we need to cater for 1 user
3. Do we have any specific moderation requests?
4. What type of content do we need to support?

The suggested questions were

1. What is the traffic volume?
2. How many friends can a user have?
3. Is this for a Mobile or Web application?
4. How should the feed be sorted?

# Architecture

A news feed has two main components

- Feed Publishing : Happens when a user publishes a post and it is populated into friend's feeds
- Feed Building: Aggregating friend's posts in reverse order

## Feed Publishing

We have a simple architecture here but here are the important components

- **Fanout Service** : This helps to populate friends feeds
  
- **Post Service** : This persists the new post and the content that the user has created in our database
  
- **Notification Service** : This notifies friends or others who are subscribed to the user making a post that a new post has been published.


![[Pasted image 20230907161422.png | 400]]

### Fanout Service

There are two main ways that we can support updates to the news feeds of users who are subscribed to our posts

1. Fanout on Write : We update their news feeds when we publish a new post
2. Fanout on Read : We update their news feeds when they request for it

*On Write* produces a snappier user experience since we generate the data ahead of time by generating the relevant updates for subscribed users. *On Read* ensures that we don't pre-generate an update for a user which might never read it. 

The best approach is therefore a hybrid one.

> **Conclusion** : For a majority of users, we use a **Fanout on Write**. For celebrities or users who have many friends/followers, we let them pull news content on-demand to avoid system overload. 

Our Fanout on Write can be implemented using the following process

1. User creates a new post and submits it to our endpoint. We start our updates by fetching friend IDs from our graph database. 
   
2. We then get friend info from the user cache and filter out friends based on user settings (Eg. if a friend has muted the user or if the user has opted to selectively share information with specific friends )
   
3. Store the new posts using a mapping of `<post_id,user_id>`. We can set these posts to expire using a TTL value.

## Newsfeed Building

![[Pasted image 20230907173033.png | 400]]

When clients want to retrieve their news feed, we can do so using the service above

1. Web servers call the News Feed Cache to compute the post ids to display
   
2. News Feed Service then rehydrates the posts with their relevant data using information from user cache and the post cache
   
3. The fully hydrated news feed is returned in JSON format back to the client for rendering. 