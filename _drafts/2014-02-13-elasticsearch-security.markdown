---
layout: post
title: Securing Elasticsearch using Finagle
category: posts
comments: true
---

[Elasticsearch](http://www.elasticsearch.org/) is the hot new kid on the noSQL-big-data block. It is a powerful tool and there are a lot of things you can do with it. But as with any system, there are some things that are not built in. For instance - security. 

What if you need HTTP access to your Elasticsearch, but want to protect it from unauthorized access? How do you add a security layer to your cluster?

One option is to use a security plugin that replaces the internal Netty server with another server that allows securing HTTP request (like [this one](https://github.com/salyh/elasticsearch-security-plugin), for example).
Using a plugin does have its caveats, though. Mainly that you are dependent on a plugin update if you want to update Elasticsearch to a newer version.

Another way to go is to create an isolation layer in front of Elasticsearch. A proxy service that tunnels the requests to Elasticsearch while enforcing security on top of them. Creating such a proxy is very easy using twitter's Finagle library.

[Finagle](http://twitter.github.io/finagle/) is: 
> an extensible RPC system for the JVM, used to construct high-concurrency servers. Finagle implements uniform client and server APIs for several protocols, and is designed for high performance and concurrency.

Constructing an HTTP proxy using Finagle is as simple as getting the request and passing it on to another Finagle HTTP client:
```scala
// our simple service - just proxy the request to ES
val service = new Service[Request, Response] {
  def apply(request: Request) = {
    _client.send(request)
  }
}
```   
