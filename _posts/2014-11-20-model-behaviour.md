---
layout: post
title: DMS Model
description: "how Ubiquitous Language shaped our domains"
tags: [DDD, Ubiquitous Language, Eventstorming]
author: andreabalducci
image:
  feature: ul-invert.jpg
  credit: PDPics
  creditlink: http://pixabay.com/it/definizione-parola-dizionario-testo-390785/
comments: true
share: true
---

## we're back
Sorry for the delay: it's a very busy time and a lot of things are happening earlier than we expected :)

We left some weeks ago with a post about [Ubiquitous Language](/2014/10/07/Ubiquitous-language.html).

Let's see and example from the domain of our simple Document Management System.

## Our users
We started our project with a round of interviews with some key roles in the organization to listen to their needs and identify how the current system was used, what they liked and disliked.

We identified two type of users:  
**Publishers**: the documents were created and published by Products Department, Legal Department, Organization Department, Board of directors, Loan Office, etc..  More than ten departments in the bank headquarters.

**Consumers**: every employee, both in headquarters and branch offices, has some interest in the documents produced by some departments, mainly based of his current employ.

## Document == File
A document is an Open Office / Ms Office / Pdf file, so we implicitly began to treat the documents as files and model a File instead of a Document.
Our "Document" was *Created*, *Deleted*, *Favorited*, *Tagged*, *Published*, *Opened*, *Analyzed*, *Protocolled*, *Versioned*, etc...
