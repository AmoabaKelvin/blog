---
title: "Improving Response Times and Reducing DB Reads"
seoTitle: "Improving Response Times and Reducing DB Reads"
seoDescription: "In this post, I will go through the mechanism I used to reduce databse read operations drastically while improving response times in ishortn.ink"
datePublished: Wed Jan 31 2024 23:51:41 GMT+0000 (Coordinated Universal Time)
cuid: cls2fysej000008la5c3o15hb
slug: improving-response-times-and-reducing-db-reads
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/iR8m2RRo-z4/upload/4f1c09a3a77a67158ae8f8f12d649f8d.jpeg
tags: software-development, system-architecture, systemdesign

---

Did you know that a 1-second delay in page load can lead to a 7% decrease in conversions?

One of the most important metrics for ensuring that the users of your software are happy with the general service is speed 🏎️. No matter how good your service is, once it is slow, users begin to look out for alternatives that will work much faster.

[My URL shortening platform](https://ishortn.ink) was getting some traction—about 1 signup daily and over 850 redirects per day—but its popularity came at a cost: a skyrocketing number of database read operations. This meant sluggish redirects and frustrated users, which was the exact opposite of what I wanted.

Response times for resolving shortened links were around 1.2 seconds, and it had already made over 44 million row reads 🤯. I got very particular about the row reads because everything is being run on a free tier, and the database has a maximum of 1B row reads. 40 million is very low compared to 1B, but I was still very concerned.

Armed with some system design knowledge, I embarked on making sure that this problem was looked into.

### Quickest thoughts: Caching

The first thing that came to mind was Caching. Sitting to reflect, I was wondering why I had not thought about including this before the earliest release. My sole reason was *premature optimization*; basically, I was thinking I was the only user, so why would I want to spend so much time thinking and implementing this?

The idea of caching made a lot of sense, and it is by far one of the most basic things to include in systems to increase response times and performance. Think about CDNs and domain name resolution caches, among other things.

Before I delve further, you might already have an idea of what the problem statement is, but then let me go ahead and state it once more!

PROBLEM STATEMENT:

> We are currently using a free tier database, which has a limit of 1B read operations. We are currently reading every link from the database everytime a new request is made, which is increasing our read operations and thus, getting us closer to the free tier limit. We want to improve response times and also reduce read operations.

### Implementation Mechanism

The next thing was how I would go about the implementation. Because getting the implementation wrong can lead to a whole lot of consequences.

* What if a link's destination is updated and the wrong data is being stored in the cache?
    
* What if a link's alias has changed? Does that mean users are redirected to a 404 page, although the link is really there?
    

Like I said, getting this wrong will lead to your users having the worst experiences. In the sections below, I will try to explain my thoughts clearly so you understand why I did certain things.

### Caching Service Used

I opted for Redis since it is the most common and the one with the most available resources on it, be it platforms offering it as a service or even supporting documentation. Since my site is built around Nextjs, I used Redis through `Upstash` service.

### Caching Strategy

Although there are numerous caching strategies, I stuck to the **write-through cache** strategy. Here, we write data to both the cache (Redis in this case) and the underlying database (MySQL) simultaneously. With this in place, what essentially happens is that, until a cache is invalidated or evicted, we will not hit `cache-miss` since the cache is readily pre-populated.

### Cache Invalidation Strategy

> Cache invalidation is a process in a computer system whereby entries in a cache are replaced or removed. \[Wikipedia\]

Sometimes, the allowed cache storage sizes might get close to filling up; when it is approaching this instance, we will have to either get rid of some of the items in the cache or clear the whole cache. This also has different ways of being done, and each has its own intention. I will list some below, then talk about the one I settled for.

* **Random Replacement:** Randomly selects an item for eviction when space is needed.
    
* **Least Frequently Used (LFU):** a strategy that prioritizes keeping items in the cache that have been accessed the most often while removing those that have been used the least
    
* **Least Recently Used (LRU):** a strategy that removes the data items that have been **accessed the least recently.**
    
* **Time To Live (TTL):** Each item in the cache is assigned a specific expiration time. When the time expires, the item is automatically evicted, regardless of its access history.
    

I ended up using the **LFU (Least Frequently Used)** strategy for our cache eviction. This is because we want to keep the most frequently used links in the cache and remove the least frequently used links from the cache. Sometimes a link might only be clicked on once; in that case, it is of no use lying in the cache and taking up precious free-tier space 🥺

### General Design

```markdown
Request -> Cache -> Database -> Cache -> Response
```

Since we are using the write-through cache approach, every time a user creates a link (probably they are going to share that link in the next instance), so we concurrently write that particular link to both the cache and the database being used.

Once a request for a particular link comes through, for example, `ishortn.ink/someAlias` I check the cache to see if we have that particular link in there. I used the aliases as the cache keys since they are unique and there will not be more than one of the same aliases in the system.

If the link exists in the cache, we go ahead, retrieve it, and also record some other things that need to be recorded, and the response is sent back to the requester.

In situations where the link is not found in the cache, it is retrieved from the database, written to the cache, and finally returned to the user.

### Outcomes

So you will realise that for the current implementation, if a new link is created and the next moment there are 600 clicks on it, it means that the only DB action we did was to just write the link to the table 🤯. Cutting down 600 read operations.

Even if the link is not found in the cache and we have to read from the DB, it means that we will just have one DB read and 599 cache reads 😎.

Not to forget, reading from the cache also means avoiding all the network overhead in retrieving something from the DB, and hence, we are able to respond to requests even much faster.

Initially, a single redirect resolution could take about 1.2s, but with all this implemented, we can literally do the same work in just shy of 200ms 🚀. Over 800 ms of work cut off!

In fact, implementing this and thinking through everything has really been fun and interesting! Now I can save precious free-tier resources and also ensure that my users are still happy with the service, so they keep referring more individuals.

Caching is really a game-changer!!!!

I hope you learned a lot from this and enjoyed reading. If you would like to inquire further, please leave a comment, and I will do my best to answer!

Happy Hacking!