---
layout: post
title: Configurations
description: "Jarvis configuration as a service"
tags: [jarvis]
author: gianmariaricci
image:
  feature: gears-configuration.jpg
  credit: wonderfulengineering.com
  creditlink: http://wonderfulengineering.com/32-amazing-gear-wallpaper-backgrounds-in-hd-for-download/
comments: true
share: true
---

## Why configurations are important
When you start working with .NET technologies it is natural to place application settings inside app.config/web.config, but this approach usually fails to be useful in long term for many reasons

- When a software is composed by multiple services, it is useful to have one common configuration file for all services that belongs to the same "suite"
- .NET config file mix application settings with .NET specific settings (es. assembly redirect)
- Xml is not the best human readable format
- Frictions for automatic deploy ([web.config transform](http://msdn.microsoft.com/en-us/library/vstudio/dd465318(v=vs.100).aspx), [SlowCheetah](http://visualstudiogallery.msdn.microsoft.com/69023d00-a4f9-4a34-a6cd-7e854ba318b5), etc)

###Resist the urge for complexity
You should resist the temptation to design an archtiecture for launching a rocket into space if you need only a better configuration manager configuration for a set of services. 

> 
Simplicity is the ultimate sophistication. [Leonardo da Vinci](http://www.brainyquote.com/quotes/quotes/l/leonardoda107812.html)
>
>Keep It Simple, Stupid. [Wikipedia](http://en.wikipedia.org/wiki/KISS_principle)

One of my greatest defects it loving architecting and this lead quite often to an unnecessary complexity, so I'm trying to change my attitude.

> 
Solving simple problem with complex solution, #FAIL.
>
Solving simple problem with simple solution, #GOOD.
>
Solving complex problem with complex solution, #ACCEPTABLE.
>
Solving complex problem with simple solution, #GENIUS.

###Keep in mind DevOps 
Releasing your sofware [must be easy and well documented procedure](http://devopsreactions.tumblr.com/post/57234308379/setting-up-a-product-following-vendors-instructions) and you should help as much as possible Operation people in understadingwhat is wrong.


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
