---
layout: post
title: Securing Elasticsearch with Finagle
category: posts
comments: true
description: Elasticsearch is the hot new kid on the NoSQL-Big-Data block. It is a powerful tool and there are a lot of things you can do with it. But as with any system, there are just some things that are not built in. For instance - security. 
---

[Elasticsearch](http://www.elasticsearch.org/) is the hot new kid on the NoSQL-Big-Data block. It is a powerful tool and there are a lot of things you can do with it. But as with any system, there are just some things that are not built in. For instance - security. 

What if you need HTTP access to your Elasticsearch, but want to protect it from unauthorized access? How do you add a security layer to your cluster?

One option is to use a security plugin (like [this one](https://github.com/salyh/elasticsearch-security-plugin), for example) that replaces the internal Netty server with another server that allows securing HTTP requests.
Using a plugin does have its caveats, though, like being dependent on a plugin update if you want to update Elasticsearch to a newer version.

Another way to go is to create an isolation layer in front of Elasticsearch. A proxy service that tunnels the requests to Elasticsearch while enforcing security on top of them. In this post I'll look into creating such a proxy in scala using twitter's Finagle library.

---

[Finagle](http://twitter.github.io/finagle/) is: 
> an extensible RPC system for the JVM, used to construct high-concurrency servers. Finagle implements uniform client and server APIs for several protocols, and is designed for high performance and concurrency.

For creating our server we'll use the cool [Twitter Server](https://github.com/twitter/twitter-server) template, which gives us out of the box some administrative APIs, statistics and other goodies.
  
Constructing an HTTP proxy using Finagle is as simple as getting the request via the Finagle server and passing it on to a Finagle HTTP client (we'll get back into the client thing in a bit):

```scala
// our simple service - just proxy the request to ES
val service = new Service[Request, Response] {
  def apply(request: Request) = {
    _client.send(request)
  }
}
```   
Next we want to add some security to our requests. This is easy to do with Finagle's composable nature. We can create a filter that allows our requests to go through only when using a magic word:

```scala
val authorization = new SimpleFilter[Request, Response] {
	def apply(request: Request, continue: Service[Request, Response]) = {
	  if ("abracadabra" == request.getParam("token")) {
	    continue(request)
	  }
	  else {
	    Future.exception(new IllegalArgumentException("Authorization enabled, wrong token"))
	  }
	}
} 
```
And now we'll compose our service to use the filter, tell Finagle to route all requests to our service, and start the server:

```scala
def main() {
  // add the filter to our service
  val protectedService = authorization andThen service
  // add routing to the service
  HttpMuxer.addRichHandler("", protectedService)
  HttpMuxer.addRichHandler("/", protectedService)
  val server = Http.serve(":8080", HttpMuxer)
  onExit {
    server.close()
  }
  Await.ready(server)
}
```

---

Until now we assumed that our proxy is connecting to Elasticsearch via HTTP, so it just takes the request and re-sends it using Finagle as an HTTP client.
But since our server is written in scala (or Java for that matter), we have the native Elasticsearch client at our disposal.

But first - why should we prefer to use the native client over Elasticsearch HTTP interface?

The reason is that we can use the Java [node client](http://www.elasticsearch.org/guide/en/elasticsearch/client/java-api/current/client.html) of Elasticsearch. This is an operational node in the cluster that functions as a client (and not as a data node). It is cluster aware, it knows the structure of the cluster and can route requests to the appropriate data nodes (the ones that hold the data we need), thus saving a potential redundant hop that is likely to happen when using HTTP. The node client also takes some load off the data nodes by doing some of the work locally (like merging results from the shards that where queried).

Using the node client in our proxy is not as straightforward and requires a bit more hacking into the core of Elasticsearch. We're going to pass the HTTP request we get directly into our Elasticsearch node client, and catch the response it generates in order to return it back.

First we extract the interfaces we need from the client - the Rest controller that handles Rest requests, and the internal Netty transport:

```scala
// build the node
val node = nodeBuilder().settings(settingsBuilder).data(false).client(true).build;
// start the node
node.start();
val internalNode = node.asInstanceOf[InternalNode];
// get the REST controller
_restController = internalNode.injector().getInstance(classOf[RestController]);
// get the transport layer
val networkService = internalNode.injector().getInstance(classOf[NetworkService])
_transport = new NettyHttpServerTransport(internalNode.settings(), networkService);
```

Now instead of re-sending the request via HTTP we'll copy it and dispatch it to the Rest controller: 

```scala
def send(request: Request) = {
	// copy the request to the ES namespace classes
	val esReq = ESClient.copyESRequest(request.httpRequest);
	// create a mock REST channel to capture the response from ES and return it
	val mockChannel = new MockRestChannel();
	// dispatch the request to the REST controller, using our own channel to receive the response
	val channel = new NettyHttpChannel(_transport, mockChannel, esReq)
	controller.dispatchRequest(new NettyHttpRequest(esReq, mockChannel), channel)
	mockChannel.future.asInstanceOf[Future[Response]];
}
```
As you can see, we provide our own mock channel to the controller. This is a Netty channel implementation that only implements the "write" method. Inside it we hold the future that is returned to Finagle. When the Elasticsearch client writes the response to the channel we update the future, so it will be returned back as a response to the original request:

```scala
class MockRestChannel extends MockChannel {
    // finagle future
	val future = new Promise[Response]
	override def write(any: Object): ChannelFuture = {
		if (any.isInstanceOf[HttpResponse]) {
			val rep = any.asInstanceOf[HttpResponse];
			// copy the response back to Netty namespace
			val response = ESClient.copyESResponse(rep);
			// update the future
			future.setValue(Response(response))
		}
		return new MockChannelFuture;
	}
}
```

So now we have a proxy server that can enforce security on HTTP requests to Elasticsearch AND use the native node client for better performance. Let the fun begin!   
(The full code for this post can be found [here on github](https://github.com/rore/finagle-es-proxy)). 