---
layout: post
title: Pub-Sub messaging with Zookeeper
category: posts
comments: true
description: Using zookeeper as a Pub/Sub messaging service for sending notifications between nodes of a distributed service.
---
When building a distributed service, there are cases where you need to broadcast messages to all nodes running your service.    
For example, if your service holds a cache of items in memory, and an operation on one of the nodes can invalidate an item, you need to notify all of the nodes to remove that item from the cache.

These types of notifications are a good use case for a [Pub/Sub](http://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern) messaging service, which usually runs on top of some kind of a [message queue](http://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern).   

But if your service is coordinated via [zookeeper](http://zookeeper.apache.org/), it can make sense to utilize zookeeper also for managing the messages between the nodes, instead of incorporating and managing a full blown message queue. In this post I'll present such an option.

For those impatient - [zkms](https://github.com/rore/zkms) is a ready to use scala library that implements the concept presented in this post. It's [available on github](https://github.com/rore/zkms) and as a Maven dependency (instructions on the project page).    
     
This comes with a **big warranty note** attached: 

Zookeeper is not built with this kind of use case in mind, and as stated in this [Netflix remark](https://github.com/Netflix/curator/wiki/Tech-Note-4) - *"ZooKeeper makes a very bad Queue source"*.   
So this solution can be valid on a medium sized cluster (a few dozens of nodes) and a rather low message rate. It should probably **not** be used when high throughput or large number of nodes are expected.

---

## General Concept

Since [zkms](https://github.com/rore/zkms) comes to answer a specific need ("cache revocation" type of notifications), it follows a few design guidelines:

- Topic based Pub/Sub.
- A message is broadcasted to a single topic.
- A consumer can subscribe to multiple topics.
- Messages are sent to online nodes only. 
- Messages are not persisted and cannot be played-back. 
- You cannot send messages to a topic if there are no subscribers for that topic.

For subscribing to messages, a consumer registers itself to a topic by creating an ephemeral node under the topic name.   
For sending messages, the producer gets all the consumers that subscribed to the topic, and places the message under each of the subscribers queue.  

## The details

The library uses 3 top zNodes: 

- `/clients` - holds the list of connected consumers.
- `/subscribers` - holds the lists of subscribed consumers per topic. 
- `/messages` - hold the lists of messages for each consumer.

Another zNode is used as a leader selection path for the *cleaner*. More on that later.

### Subscribing

Subscribing to messages under a topic consists of the following steps:

1. The consumer creates an ephemeral node with its consumer ID under `/clients`. This is used by the cleaner to remove message queues for disconnected consumers.
2. The consumer creates an ephemeral node with its consumer ID under `/subscribers/[topic]`.
3. The consumer creates a watcher on `/messages/[consumerID]/[topic]`. The producer will place the topic messages under this node.

**Unsubscribing** from a topic means deleting the topic messages zNode and the subscription zNode for the consumer.  

### Broadcasting

To broadcast a message to a topic the producer:

1. Gets all the child nodes of the topic subscribers path (`/subscribers/[topic]`). If there are no subscribers, an error is returned.
2. For each subscriber, create a sequential persistent node under its message queue node (`/messages/[consumerID]/[topic]`). The node value is the message.

### The Cleaner

Each consumer generates its random consumer ID when it's created. Since consumers can go up and down frequently, and since the consumer messages node cannot be ephemeral (ephemeral nodes are not allowed to have children), we can end up with a lot of messages zNodes for dead consumers that are no longer valid.

To handle that, one of the zkms instances (either producer or consumer) functions as a *cleaner* (it's elected via a leader selection recipe).

The cleaner wakes up every now and again, and looks for messages zNodes that do not have a corresponding zNode under the `/clients` path (since the consumer registers an ephemeral node there, it will be gone when it gets disconnected). It deletes (recursively) message nodes with no connected consumers.

## Performance

Since broadcasting messages means creating a zNode for every consumer which is subscribed to the topic, the performance of the publisher degrades linearly with the number of topic subscribers.

In a not very accurate test conducted on my local development machine, a single publisher managed to push around 1000 messages per second to a topic with a single subscriber. When increasing the number of subscribers to 10 the message rate went down to ~250/sec. With 20 subscribers it was ~125/sec. 
 
So the warning is in order again - this solution will work when there are not too many subscribers and a reasonable message rate. It will not scale! Use with care.