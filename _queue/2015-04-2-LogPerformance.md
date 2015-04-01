tat cena---
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

Rebuilding all read-models (projections) in an Event Source Based application can be slow because we are processing hundreds of thousands of events with several projections. This is one area where having a boost in performance is crucial and it is where I want to understand **how much our logging infrastructure impact on performance**.

For this scenario I have a small batch of events (~4k) and log4net logging level at DEBUG level; this will generates ~35k log messages in the system in this reference situation.

## First run

Here is what happened with the first run. 

![First Run - no log enabled](/images/posts/logperformance/first.jpg)

This is a low resolution image, but it represents how much time I need to rebuild all projections with no log enabled. *The Y-Axis represents how many events are to be dispatched for each projection-slot, X-Axis represents time. *

This is the reference scenario, now I enabled logging at DEBUG level, both for our MongoBufferedAppender and for ColoredConsoleAppender. Even if the image is a low resolution one and we have no legend on X-Axis, you could be surprised that this is enough to verify that *logging is really killing performances*.

## What happens with all logs enabled

The situation really changed a lot: 

![Second Run - ALL logs enabled](/images/posts/logperformance/second.jpg)

Even if you have no X-Axis legend, it is really evident that performances are dropped down a lot. We do not need exact measurement, the situation is so different that it is really evident from a quick look that **enable logging is slowing down everything**.

We have also a really strange behaviour, most of the projections consumes events lineraly, three slots are really slower (and this can be normal), but when the other slots finished rebuild, the three slow slots speed up considerably. 

**Believe it or not, the reason is, ColoredConsoleAppender**. We have services running in Console Applications with [TopShelf](https://github.com/Topshelf/Topshelf) to run as a service in production environment, and we use console appender to have a visual clue of what is happening in each service during debug, when services are run as standard console apps.

## Stopping ColoredConsoleAppender

The next run is done simply setting threshold level of ColoredConsoleAppender to INFO instead of DEBUG. Since we have no more than 500 logs of level INFO or above, it is like disabling it completely. Here is the result.

![Third Run - only mongodbappender](/images/posts/logperformance/third.jpg)

You can notice that with only mongo appender the whole rebuild is really faster. Again, you do not need exact measurement, the impact of disabling ColoredConsoleAppender is so great that it is evident that Logging to console has a huge impact on performances. 

Actually I never liked too much console appender in log4net for various reason.

- It slows down your application
- It is difficult to read logs, console app have a limited buffer space and logs disappears quickly.
- Log disappear when the program is closed
- You have no way to query log, you only have a huge sequence of text
- If the program crashes usually you see a red log with the exception and the console app immediately closes befoure you can read what happened.

All these consideration lead to an obvious conclusion: Console Appender is not useful, the only purpose is showing you that the program **is doing something and it is not hung somewhere**

	If you want to use logging infrastructure to have an idea of **progress** of your application, please convince yourself that this is not a good idea. Using logs to verify that the application is *doing something* has little utility.

After disabling ColoredConsoleAppender it is time to a final run.

## Using MongoDbAppender with writer on a dedicated thread and LooseFix

**We can further improve performance if you know how log4net works internally.** Our MongoDbAppender inherits from BufferingAppenderSkeleton and you can read on my english blog why it slows down performances: [BufferingAppenderSkeleton performance problem](http://www.codewrecks.com/blog/index.php/2015/03/27/bufferingappenderskeleton-performance-problem-in-log4net/).

Armed with this knowledge I've done a final run using LooseFix and asking mongo appender to use a dedicated thread to save logs and here is the result.

![Fourth Run - mongodbappender with LooseFix and dedicated thread](/images/posts/logperformance/fourth.jpg)

Even if the image is small, you can verify that this image is quite similar to the first one. To verify how much performance we are loosing, we now need to measure exactly because we cannot tell anymore from a quick look to the image. MongoDbAppender have several advantages over ColoredConsoleAppender

- It is durable; logs are maintained after the application is closed
- You can query to find logs with lots of conditions (even regular expression)
- If you constantly read latest 10 logs and total count from mongo you can have an **idea of *progress*** (this is not a perfect solution, but at least you can use for this purpose if you want).

## What we learned

This example give us some guidelines on how to handle log in production:

	Logging is useful and the more information you logs, the easier is to find problems. The golden rule is: measure performance penalties in production, use only a single appender that persists logs in a durable medium that allow searching and you are done.

Gian Maria.
