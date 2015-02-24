---
layout: post
title: Why Configurations
description: "Why spent time on configuration service"
tags: [configuration, devops, refactoring]
author: gianmariaricci
image:
  feature: gears-configuration.jpg
  credit: wonderfulengineering.com
  creditlink: http://wonderfulengineering.com/32-amazing-gear-wallpaper-backgrounds-in-hd-for-download/
comments: true
share: true
---

## Why spent time on configuration
**Question**: We have this super interesting software with Mongo + Elastic Search + CQRS + Event Sourcing but we spent time doing refactoring on Configuration Service instead of working on most intesting stuff. Why?

**Answer**: From an agile point of view, the most important things to do are the one on the top of the Backlog and the Backlog is not managed by programmers. Having configurations inside config file is annoying for automatic deploy. We think automatic (or easy and fast manual) deploy has a key importance in our process.

###Why automatic deploy is important
You can create unit test, automatic acceptance test, or doing manual tests, but the ultimate truth is that software is not really tested until it is not in production.

There are lots of reason for this, but these can be syntethized in 

>
Real Users, with real data, using software for real.
>

Until the code is only in your source control system, it is not tested and we feel a little bit *dirty*, because we are not 100% confident of what will happen when the new code will go in production. 

The more you wait, the more code you will deploy in one big chunk, the more problems you will have. The more code you deploy, most difficult is understanding where you indroduced the bug, because you have more code to look at and more commit to check with git bisect. 

>
The ultimate goal is XCopy (almost) deployment
>

How beautiful is simply taking new artifacts from your Continuous Integration server and simply launch a script that copies new artifacts over the new one to update you version? One of the greates friction for this is handling configuration when they are buried in multiple config files, splitted by various services, and between multiple instances of multiple server working on different machines.

###Risk of configurations in files

There are lots of risks in managing configurations with standard config files.
