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

<figure>
  <a href="/images/jarvis-architecture.jpg">
    <img src="/images/jarvis-architecture.jpg" alt="Jarvis architecture">
  </a>
</figure>
