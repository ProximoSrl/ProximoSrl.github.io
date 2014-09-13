---
layout: post
title: What is Jarvis?
description: "The product we are building right now"
tags: [jarvis, domain, intro]
author: andreabalducci
image:
  feature: knowledge-channel.jpg
  credit: Jon Mannion
  creditlink: https://flic.kr/p/6N3Dso
comments: true
share: true
---

## File > New > Intranet

This is the first step of this [journey](/about-this-blog/), we want to tell you what we have built so far.

### One year ago we started an "Intranet 2.0" project for a small bank.

The project main goal was to improve the ease of access to key informations, avoid personal (and sometimes) out of sync document folders, on user's desktop.

We found a ["Yahoo Directory"](https://dir.yahoo.com) of about 4.000+ relevant documents (250.000+ pages).

To simplify document retrieval, every office developed a custom taxonomy to catalog corporate documents.

<blockquote>
In the beginning there was Chaos,<br/>
And within this Chaos was Power,<br/>
Great Power without Form
</blockquote>

Forgive Google and try to search the internet browsing the ["Yahoo Directory"](https://dir.yahoo.com).. you got it!

Our first goal was: no more phone calls to "knowledge owners" to find "that document" or endless filesystem style browsing in search of relevant informations.

So our first domain was a [DMS](http://en.wikipedia.org/wiki/Document_management_system).

<figure>
<a href="https://www.flickr.com/photos/kiarras_marinero/8455661691" title="library by kiarras, on Flickr"><img src="https://farm9.staticflickr.com/8532/8455661691_577e00039c_z.jpg" width="640" height="426" alt="library"></a>
  <figcaption><a href="https://flic.kr/p/dTctCx" title="library by kiarras, on Flickr">library by kiarras, on Flickr</a>.</figcaption>
</figure>

Then we moved on:

* Support Ticketing with Knowledge Base,
* Corporate address book,
* Home Banking workflow,
* Point Of Sale workflow,
* Resource scheduling system,
* Corporate bookmarks,
* Corporate chat & file exchange,
* Active Directory sync,
* Organisation chart,
* IT asset inventory,
* Analytics.

Some domains have been implemented with [CQRS](http://martinfowler.com/bliki/CQRS.html)+[ES](http://martinfowler.com/eaaDev/EventSourcing.html), others with CRUD.

Next post I'll cover the current architecture and our dev stack.

Stay tuned!
