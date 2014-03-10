---
layout: post
title: Building a social music service - the technology behind serendip.me
category: posts
comments: true
description: The technology and architecture behind serendip.me - a social music service built with scala, akka, MongoDB and Elasticsearch. 
---

During 2011-2013 I had the pleasure of working as Chief Architect for [serendip.me](http://serendip.me), helping to build it from the ground up together with [Sagee](http://il.linkedin.com/in/sageebz) and [Asaf](http://il.linkedin.com/in/asafatzmon), Serendip’s founders.   
I wanted to share some of the experience of building serendip’s backend - the technologies we used, architecture and scaling considerations - since personally I always find it very insightful to hear about real-life start-up experiences, especially when trying to create a new venture. 

---

[serendip.me](http://serendip.me) is a social music service that helps people discover great music shared by their friends, and also introduces them to their “music soulmates” - people outside their immediate social circle that shares a similar taste in music.

Serendip is running on AWS and is built on the following stack: [scala](http://www.scala-lang.org/) (and some Java), [akka](http://akka.io/) (for handling concurrency), [Play framework](http://www.playframework.com/) (for the web and API front-ends), [MongoDB](http://www.mongodb.org/) and [Elasticsearch](http://elasticsearch.org/).

### Choosing the stack

One of the challenges of building serendip was the need to handle a large amount of data from day one, since a main feature of serendip is that it collects every piece of music being shared on Twitter from public music services. So when we approached the question of choosing the language and technologies to use, an important consideration was the ability to scale. 

The JVM seemed the right basis for our system as for its proven performance and tooling. It's also the language of choice for a lot of open source system (like Elasticsearch) which enables using their native clients - a big plus.    
When we looked at the JVM ecosystem, scala stood out as an interesting language option that allowed a modern approach to writing code, while keeping full interoperability with Java. Another argument in favour of scala was the akka actor framework which seemed to be a good fit for a stream processing infrastructure (and indeed it was!). The Play web framework was just starting to get some adoption and looked promising. Back when we started, at the very beginning of 2011, these were still kind of bleeding edge technologies. So of course we were very pleased that by the end of 2011 scala and akka consolidated to become [Typesafe](http://typesafe.com/), with Play joining in shortly after.

MongoDB was chosen for its combination of developer friendliness, ease of use, feature set and possible scalability (using auto-sharding). 
We learned very soon that the way we wanted to use and query our data will require creating a lot of big indexes on MongoDB, which will cause us to be hitting performance and memory issues pretty fast. So we kept using MongoDB mainly as a key-value document store, also relying on its atomic increments for several features that required counters.   
With this type of usage MongoDB turned out to be pretty solid. It is also rather easy to operate, but mainly because we managed to *avoid* using sharding and went with a single replica-set (the sharding architecture of MongoDB is pretty complex). 

For querying our data we needed a system with full blown search capabilities. Out of the possible open source search solutions, Elasticsearch came as the most scalable and cloud oriented system. Its dynamic indexing schema and the many search and faceting possibilities it provides allowed us to build many features on top of it, making it a central component in our architecture.

We chose to manage both MongoDB and Elasticsearch ourselves and not use a hosted solution for two main reasons. First, we wanted full control over both systems. We did not want to depend on another element for software upgrades/downgrades. And second, the amount of data we process meant that a hosted solution was more expensive than managing it directly on EC2 ourselves. 

### Some numbers

Serendip’s “pump” (the part that processes the Twitter public stream and Facebook user feeds) digests around 5,000,000 items per day. These items are passed through a series of “filters” that detect and resolve music links from supported services (YouTube, Soundcloud, Bandcamp etc.), and adds metadata on top of them. The pump and filters are running as akka actors, and the whole process is managed by a single m1.large EC2 instance. If needed it can be scaled easily by using akka’s remote actors to distribute the system to a cluster of processors.

Out of these items we get around 850,000 valid items per day (that is items that really contains relevant music links). These items are indexes in Elasticsearch (as well as in MongoDB for backup and for keeping counters). Since every valid item means updating several objects, we get an index rate of ~40/sec in Elasticsearch.    
We keep a monthly index of items (tweets and posts) in Elasticsearch. Each monthly index contains ~25M items and has 3 shards. The cluster is running with 4 nodes, each on a m2.2xlarge instance. This setup has enough memory to run the searches we need on the data.

Our MongoDB cluster gets ~100 writes/sec and ~300 reads/sec as it handles some more data types, counters and statistics updates. The replica set has a primary node running on a m2.2xlarge instance, and a secondary on a m1.xlarge instance. 

### Building a feed

When we started designing the architecture for serendip’s main music feed, we knew we wanted the feed to be dynamic and reactive to user actions and input. If a user gives a “rock-on” to a song or “airs” a specific artist, we want that action to reflect immediately in the feed. If a user “dislikes” an artist, we should not play that music again.   
We also wanted the feed to be a combination of music from several sources, like the music shared by friends, music by favorite artists and music shared by “suggested” users that have the same musical taste.   
These requirements meant that a “[fan-out-on-write](http://www.quora.com/What-is-the-best-storage-solution-for-building-a-news-feed-MongoDB-or-MySQL)” approach to feed creation will not be the way to go. We needed an option to build the feed in real-time, using all the signals we have concerning the user. The set of features Elasticsearch provides allowed us to build this kind of real-time feed generation. 

The feed algorithm consists of several “strategies” for selecting items which are combined dynamically with different ratios on every feed fetch. Each strategy can take into account the most recent user actions and signals. The combination of strategies is translated to several searches on the live data that is constantly indexed by Elasticsearch. Since the data is time-based and the indexes are created per month, we always need to query only a small subset of the complete data.    
Fortunately enough, Elasticsearch handles these searches pretty well. It also provides a known path to scaling this architecture - writes can be scaled by increasing the number of shards. Searches can be scaled by adding more replicas and physical nodes.

The process of finding “music soulmates” (matching users by musical taste) is making good use of the faceting (aggregation) capabilities of Elasticsearch. 
As part of the constant social stream processing, the system is preparing data by calculating the top shared artists for social network users it encounters (using a faceted search on their shared music).    
When a serendip user gives out a signal (either by airing music or interacting with the feed), it can trigger a re-calculating of the music soulmates for that user. The algorithm finds other users that are top matched according to the list of favorite artists (which is constantly updated), weighing in additional parameters like popularity, number of shares etc. It then applies another set of algorithms to filter out spammers (yes, there are music spammers…) and outliers.   
We found out that this process gives us good enough results while saving us from needing additional systems that can run more complex clustering or recommendation algorithms. 

### Monitoring and deployment

Serendip is using [ServerDensity](http://www.serverdensity.com/) for monitoring and alerting. It’s an easy to use hosted solution with a decent feature set and reasonable pricing for start-ups. ServerDensity natively provides server and MongoDB monitoring. We’re also making heavy use of the ability to report custom metrics into it for reporting internal system statistics.    
An internal statistic collection mechanism collects events for every action that happens in the system, and keeps them in a MongoDB collection. A timed job reads those statistics from MongoDB once a minute and reports them to ServerDensity. This allows us to use ServerDensity for monitoring and alerting Elasticsearch as well as our operational data. 

Managing servers and deployments is done using Amazon Elastic Beanstalk. Elastic Beanstalk is AWS’s limited PaaS solution. It’s very easy to get started with, and while it’s not really a full featured PaaS, its basic functionality is enough for most common use cases. It provides easy auto-scaling configuration and also gives complete access via EC2.    
Building the application is done with a [Jenkins](http://jenkins-ci.org/) instance that resides on EC2. The Play web application is packaged as a WAR. A [post-build script](https://github.com/rore/beanstalk-upload) pushes the WAR to Elastic Beanstalk as a new application version. The new version is not deployed automatically to the servers - it’s done manually. It is usually deployed first to the staging environment for testing, and once approved is deployed to the production environment. 

### Takeaways

For conclusion, here are some of the top lessons learned from building serendip, not by any special order.

1. **Know how to scale**. You probably don’t need to scale from the first day, but you need to know how every part of your system can scale and to what extent. Give yourself enough time in advance if scaling takes time.
2. **Prepare for peaks**. Especially in the life of a start-up, a single lifehacker or reddit post can bring your system down if you’re always running at near top capacity. Keep enough margin so you can handle a sudden load or be ready to scale really fast.
3. **Choose a language that won’t hold you back**. Make sure the technologies you want to use have native clients in your language, or at least actively maintained ones. Don’t get stuck waiting for library updates.
4. **Believe the hype**. You want a technology that will grow with your product and will not die prematurely. A vibrant and active community and some noise about the technology can be a good indication for its survival. 
5. **Don’t believe the hype**. Look for flame posts about the technology you’re evaluating. They can teach you about its weak points. But also don’t take them too seriously, people tend to get emotional when things don’t work as expected.
6. **Have fun**. Choose a technology that excites you. One that makes you think “oh this is so cool what can I do with it”. After all, that’s (also) what we’re here for.