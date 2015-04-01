 cena---
layout: post
title: Cost of logging in your architecture
description: "Have you ever measured how much log code impact on your application performance? If you are using log4net you could learn something interesting profiling the code."
tags: [testing, csharp]
author: gianmariaricci
image:
  feature: logperformanceheader.jpg
  credit: Gian Maria Ricci
  creditlink: http://www.codewrecks.com/
comments: true
share: true
---

## Rebuilding everyting in a Event Source Based application

Rebuilding all read-models (projections) in an Event Source Based application can be slow, because we are processing hundreds of thousands of events with several projection. This is one area where having a boost in performance is crucial and it is where I want to understand how much my logging infrastructure is slowing down everything.

In this situation I have a small batch of events (~4k) and I decided to keep log4net logging level to DEBUG, this will generates ~35k logging messages in the system.

## First run

Here is what happened with the first run. 

![First Run - no log enabled](/images/posts/logperformance/first.jpg)

This is a low resolution image, but it represents how much time I need to rebuild all projections with no log enabled. The Y-Axis represents how many events are to be dispatched for each projection-slot, X-Axis represents time. 

This is the reference scenario, now I enabled logging at DEBUG level, both for our MongoBufferedAppender and for ColoredConsoleAppender.

## What happens with all logs enabled

The situation really changed a lot: 

![Second Run - ALL logs enabled](/images/posts/logperformance/second.jpg)

Even if you have no X-Axis legend, it is really evident that performances are dropped down a lot. We have also a really strange behaviour, most of the projection consumes events in a linear way, but three of them seems to be really slower, and when the other projection finished rebuild, the three slow projection speed up considerably.

Believe it or not, the reason is, ColoredConsoleAppender. We have services running in Console Applications that uses [TopShelf](https://github.com/Topshelf/Topshelf) to run as a service, and we use console appender to have a visual clue of what is happening in each service.

## Stopping ColoredConsoleAppender

The next run is done simply setting threshold level of ColoredConsoleAppender to INFO instead of DEBUG. Since we have no more than 500 log of INFO or above, it is like disabling it completely. Here is the result.

![Third Run - only mongodbappender](/images/posts/logperformance/third.jpg)

You can notice that with only mongo appender the whole rebuild is really faster. Logging to console has a huge impact on performance and should be done only for some logs. Here is why I do not like console logger

- It slows down your application
- It is difficult to read, console app have a limited buffer space 
- If the program crashes usually you see a red log with the exception and the console app immediately closes. 

## Using MongoDbAppender with writer on a dedicated thread and LooseFis

You can read details on this subject on my english blog: [performances of BufferingAppenderSkeleton]() in log4net. Actually I've done a final run using LooseFix and asking mongo appender to use a dedicated thread to save logs and here is the result.

![Fourth Run - mongodbappender with LooseFix and dedicated thread](/images/posts/logperformance/fourth.jpg)

Even if the image is small, you can verify that this image is quite similar to the first one. To verify how much performance we are loosing, we need to measure with the code because with a quick look we cannot tell if using mongodbappender really slows down the application.

Gian Maria.
