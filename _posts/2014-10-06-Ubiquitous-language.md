---
layout: post
title: Ubiquitous Language
description: "how Ubiquitous Language shaped our domains"
tags: [DDD]
author: andreabalducci
image:
  feature: ul-invert.jpg
  credit: PDPics
  creditlink: http://pixabay.com/it/definizione-parola-dizionario-testo-390785/
comments: true
share: true
---

## What is
<blockquote>
Programmers should speak the language of domain experts to avoid
miscommunication, delays, and errors. To avoid mental translation between domain
language and code, design your software to use the language of the domain.
Reflect in code how users of the software think and speak about their work.
One powerful approach is to create a domain model.
<br/><br/>
The ubiquitous language is a living language. Domain experts influence design
and code and the issues discovered while coding influence domain experts.
When you discover discrepancies, it's an opportunity for conversation and joint
discovery. Then update your design to reflect the results.
</blockquote>
*[The Art of Agile Development](http://www.jamesshore.com/Agile-Book/ubiquitous_language.html)*


## It's all about words?
Some time ago I had the pleasure to meet with some friends for our
[GUISA meeting](http://www.eventbrite.it/e/guisa-meeting-1-tickets-2972429617).

[Andrea Saltarello](https://twitter.com/andysal74) gave the
"Ubiquitous Language Smackdown" talk in which he shared his experience.
We made a very similar experience ourself (same terms, same communication issues
with our customer), but I was firmly convinced that this approach could be used
only on projects and was not suitable for products where domain experts are our
clients (we craft business software for SME and Public Sector).

Based on our experience every client speaks it's own language;
it can depends on the market, the geographical location (even within the same
country) and company size.

Among our products there's a [Manufacturing execution system](http://en.wikipedia.org/wiki/Manufacturing_execution_system);
if we talk to a mechanical parts manufacturer we can refer to "Bill of Materials",
but if we talk to a fashion manufacturer and we say "Bill of Materials" the games
are over :"your product is not for us, we don't deal with BOM".
Their bill of materials (IT: distinta base) is "Scheda tecnica"; I don't know
how to translate with the correct meaning in english, maybe in english there's
not this distinction. If we talk with an PCB assembly company, "Scheda tecnica"
means "Product Sheet".

With all these distinctions how can we model our software in an Ubiquitous way?


## No it's all about meanings

<blockquote>
There's a sign on the wall but she wants to be sure<br/>  
Cause you know sometimes words have two meanings<br/>
<br/>
Stairway to Heaven - Led Zeppelin
</blockquote>
