---
layout: post
title: How we built our own .NET microservices framework (and made it open source along the way)
category: posts
comments: true
description: How and why we built Microdot, a home-grown microservices framework in .NET, and made it open source.  
---
There are some things in life that everyone tells you not to do.   
Sometimes you still do them.    
One of these things is building your own framework. (Don’t you just hate it when your parents tell you again and again - don’t build your own framework?).

Well, we have some excuses. 

When we (at Gigya) embarked on our microservices journey two years ago, the .NET landscape was somewhat barren. (Why .NET? We had a lot of application code in .NET we wanted to use and reuse. Our developers where .NET people. We had a lot of experience already. Moving away from that was not really a reasonable option).   
Java and other common languages had multiple libraries, frameworks and technologies for building microservices. But the .NET open source universe was not really expanding (there were some [interesting](http://www.aaronstannard.com/the-profound-weakness-of-the-net-oss-ecosystem/) [discussions](http://code972.com/blog/2016/01/93-open-source-and-net-its-not-picking-up) about that). So in a way, we didn’t have too much of a choice. 

But the nice thing about being “forced” to make such a choice, is that it makes the journey very interesting. It gives you a chance to learn what options are out there, how people approach the problem of microservices and what various design decisions they make. It allows you to select what fits your own use case and constraints. It also introduces multiple never ending discussions… That eventually you need to end.

One of the essential questions was a meta-discussion about the direction of our path. Do we need a microservices framework at all? Alternatively we can define a set of basic rules (HTTP transport, some common practices) and let everyone take their own approach (and maybe technology) in writing their microservice.    

This polyglot approach is valid, and is considered sometimes one of the benefits of a microservices environment. But there’s overhead to consider when working in multiple languages, technologies or design choices. Both in terms of operations (deployment, monitoring, infrastructure) and of people and knowledge. As a not-too-big development organization, this was an overhead we didn’t want to introduce at the time.        
We decided to have a common framework for our microservices that will dictate some of the basic patterns of how we see our microservices working. Solve the common problems in a single place, and let everyone reuse it. That was our goal. 

Another reason was a non-trivial technology choice we took as the common ground of our new microservices. This was a new technology that was just open sourced by Microsoft Labs - [Orleans](http://dotnet.github.io/orleans/), a Virtual Actors framework. I won’t go too much into Orleans here (I [talked](https://vimeo.com/190911340) about it on other occasions). Since we are handling high loads of state-sensitive traffic, we believed Orleans and the Virtual Actors model gives a good way of handling problems of distributed concurrency that we knew we need to handle. Orleans gave us a productive and developer friendly way of building such high scale services. We wanted our microservices framework to play nicely with Orleans and solve the common hosting and deployment scenarios, so that putting up a new Orleans base microservice will be a common and simple routine. Hopefully we managed to do that. 

Another organizational question was how to manage the development of the framework. Will it be a common library that all teams work on and improve as they go along? Or will we have a dedicated team that will be responsible for this common framework? There are pros and cons for each approach. We preferred clear ownership and focusing the effort. So we created a dedicated infrastructure team that is responsible for the development of the common microservices framework.    
It was not a smooth process, of course. It creates a situation of dependencies between teams (application teams might need to wait for a feature to be developed in the infrastructure). But we tried to put in place processes to identify needs and plan accordingly. We also encouraged an internal “open source” approach, where each team can make changes to the common framework on its own and create a pull request that is reviewed by the infrastructure team. And as the framework matured and stabilized, those inter-dependency issues became less problematic.

The end (“end” might not be the right terminology, it’s still a work in process) result is a pretty robust framework with some interesting features. We selected an *RPC-like* approach for our inter-service communication, modeling it after Orleans own inter-grain communication pattern. We added *transparent client side response caching* to accelerate responses when possible. We support *distributed tracing*, *client side load balancing*, *health checks*, a *hierarchical file based configuration system*, and more. Our framework implements a lot of the patterns that are needed for building up microservices. 

Since there are less than a handful open source frameworks for microservices in .NET, we thought that our own home grown framework might be a nice addition. We believe its feature set is rather unique, and it is production proven.    
But getting it to be ready to open source took some time. First, you need to get buy-in from the developers working on it. It took time and the maturity of the framework until people agreed that this is something that is worth putting out.    
Then there’s some time investment needed. We had to refactor and make the code more modular so we can take out internal things and make the core framework general purpose and generic. Make our internal framework use the open source version. Write some documentation. We allocated time and our developers did a great job making it ready for open source.    

It was a nice moment switching the privacy setting on the repository in github, making Microdot open to the world - [https://github.com/gigya/microdot](https://github.com/gigya/microdot).