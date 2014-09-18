---
layout: post
title: Architecture
description: "Jarvis current architecture"
tags: [neventstore, angularjs, mongodb, elasticsearch, bus, webapi]
author: andreabalducci
image:
  feature: old-metro-map.jpg
  credit: Alexander Baxevanis
  creditlink: https://flic.kr/p/bm1S9j
comments: true
share: true
---

## Jarvis Dev stack
In the previous post we introduced Jarvis, in this we'll talk about the development stack and the architecture.

The client is an Html5 AngularJs Single Page Application,
our backend is mostly c#.

<figure>
  <a href="/images/jarvis-building-blocks.jpg">
    <img src="/images/jarvis-building-blocks.jpg" alt="Jarvis building blocks">
  </a>
</figure>

The core domains are implemented in eventsourcing and cqrs on top of [NEventstore](http://neventstore.org) with a custom implementation of CommonDomain.  
For [NEventstore v5](http://neventstore.org) we needed a better implementation of [Mongo Persistence Engine](https://github.com/NEventStore/NEventStore.Persistence.MongoDB) so I started pushing our fixes on Github.. some pull requests after I was invited to join the team (thank you [Damian](https://twitter.com/randompunter)).  
That was our first giveback to the dev community, then I patched / enhanced the majority of the core libraries we used in this project.  

**That's (imho) how open source is supposed to work, less complains and more pull requests.**

Jarvis core domains are hosted in few ([Topshelf](https://github.com/Topshelf/Topshelf) based) windows services  talking each other with [Rebus](https://github.com/rebus-org/Rebus) and [MSMQ](http://msdn.microsoft.com/en-us/library/ms711472(v=vs.85).aspx).

The [webapi](http://www.asp.net/web-api) services are hosted in IIS and secured with integrated authentication.

## Data flow

### Frontend api
Jarvis data flow is "realtime async": our frontend api accepts and validate user commands, translate them in domain commands and push them to our commandbus for delivery.

### Domain services
The destination worker handle the command, with automatic retry and failover, invoking the corresponding aggregate methods, it's a sort of RPC over service bus: there is not a strict one to one mapping between commands and methods.

The target aggregate emits events in response to method calls to change its state, the events are persisted in NEventStore waiting for consumers: [projections](http://cqrs.wikidot.com/doc:projection)
and
[process managers](http://msdn.microsoft.com/en-us/library/jj591569.aspx).

### Process managers
Every [process manager](http://msdn.microsoft.com/en-us/library/jj591569.aspx) place a subscription on Rebus for the events it is interested in.
To be more accurate a process manager can subscribe to events and messages; a message doesn't have domain semantic and is exchanged with external high latency services.
The process manager react to events sending new commands and messages.

### High latency services
We need to rely on external services for text extraction, document conversion and analisys; all the cpu intensive and high latency tasks are hosted by a service worker installed on one or more machines with a simple round robin routing for task distribution.

### Projections
Our [projections](http://cqrs.wikidot.com/doc:projection) service works in pull mode; projections are grouped by affinity; every group has a subscpription to a polling client on the evenstore and run in its own thread.
Our querymodel is denormalized on [MongoDb](http://www.mongodb.org) and [Elasticsearch](http://www.elasticsearch.org).

### Push notifications
Query model updates are pushed to subscribers with [SignalR](http://signalr.net) and processed by AngularJs for interface updates.

<figure>
  <a href="/images/jarvis-architecture.jpg">
    <img src="/images/jarvis-architecture.jpg" alt="Jarvis architecture">
  </a>
</figure>

## What's next
This architecture worked well for our first customer, we have over 100 concurrent users, databases, frontend api and worker services on the same vmware box.
Cpu is mostly between 1-2%, ram consumption flat and the whole roundript usually takes few milliseconds.

Rebuilding all the 50+ projections from 350k+ commits takes almost 10 minutes (we are single thread), we need to improve in this area.

Our querymodel is about 4gb, we expect to grow (at least) by a [100x factor](/about-this-blog/) in the next two months.

We want to split our domains in isolated services and remove the friction for every deploy: we deploy twice a week, some weeks every day (or twice a day).

To achieve this goal [Gian Maria](/about/gianmariaricci/) is working on a configurations dispatcher service and getting rid of all the connection strings and application settings palced in the web/app.config.

More in the next post.
