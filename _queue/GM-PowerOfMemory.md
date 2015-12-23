---
layout: post
title: Power of RAM when you need performance
description: "How working in RAM can increase performance"
tags: [DDD, EventSourcing]
author: gianmariaricci
image:
  feature: gears-configuration.jpg
  credit: wonderfulengineering.com
  creditlink: http://wonderfulengineering.com/32-amazing-gear-wallpaper-backgrounds-in-hd-for-download/
comments: true
share: true
---

### Start with a picture
Following picture represent timing during a full rebuild for various situation I'm going to explain in this post. When it is time of a full Rebuild you want to achieve maximum performance, because you need to process hundred of thousands, or millions of events, and you want it to finish as soon as possible.

- Folder_Id
- ParentFolder_Id
- Name

### First line: standard rebuild
The first line represents a standard rebuild, where the two graphs represents two distinct type of Projections. The left one is the counter for default projection (all except one) the right one represent counter for projections that uses direct access to Mongo. 

What I mean by *"direct access"* will be clear in the following paragraph, when I explain the concept of **Nitro Rebuild**.

### Second line: Nitro rebuild
We are in 2015, and RAM does not costs so much. It is standard to have machine with 8/16 GB RAM, and thanks of virtualization we can even afford a situation where a machine has a boost of memory for a small amount of time. This means that when it is time of a full rebuild, we have no problem if we use several GB of ram to process everyting in RAM.

The usual event pattern for a projection is:

1. An event arrive
2. Load from Read Model Database corresponding record
3. Modify accordingly to Event 
4. Save again record inside the database

Given this pattern, and the fact that a single readmodel object is affected by many events, the situation will be really faster if we use an in memory Dictionary with the Id of the object as key and the object as value. In this scenario we avoid loading data from an external database, and the overall speed of the rebuild is really really faster.

For this reason we create an abstraction for MongoCollection, and this abstraction allows to work in RAM for the rebuild and works with standard MongoCollection during normal operation. This approach creates a problem, for specific projection where we really need to use Mongo Specific operators, we cannot use this abstraction so we have a slower rebuild. To avoid problem, in this version the In Memory operation is available only during the rebuild. 

This is the reason why we have two graph, the one on the right belongs to a specific projection that uses heavily Mongo specific Update operator to speedup handling of event that needs to update multiple records. For those specific projection working in memory could give a faster rebuild, but slower overall speed. 

The typical operation that benefit for direct Mongo access is: I need to push a tag for all document with a specific condition. With direct mongo access is a simple Update with Push, while operating in memory forces us to load every document from db, add the tag and then save it again.

This gives us a good balance, quite all the projections can operate in memory, small set of specific and hightly specialized collections can use direct access to mongo.

Once the rebuild is finished all the in memory databases are flushed to Database engine in Batch,and this is really really faster than modify a record in db for each event that is dispatched to the projection.

### Third line: In memory index
Examining the second line you can notice that the rebuild is really really faster, but at the very beginning the slope of the graph is quite equal to first line. This means that at the start of the rebuild we have a situation where operating in memory is not really faster than operating directly on Database. How can this be possible?

Thanks to Metrics.NET we have the exact total time for every type of dispatched event / projection so we are immediately able to identify the projection that is going slow. 

At the very beginning of EventStream there is the initial load of about 5000 documents into Jarvis, and there is a specific projection that have a slightly different event pattern

1. An event arrive
2. Load from Read Model Database all records that have a property equal to a specific value
3. Modify all these objects 
4. Save again records inside the database

This happens because a projection need to find all the revisions that belongs to a given document, so it is issuing the instruction

 _revisions.FindAndModify(e, x => x.DocumentId == e.AggregateId, x => x.ProtocolNumber = e.ProtocolNumber);

This situation create a full scan of the entire In Memory dictionary and this can be slower than accessing a Database, because database has indexing.

For this reason we added a nice feature for Nitro mode, you can use a specific API to find a document given a specific property.

_revisions.FindAndModifyByProperty(e, x => x.DocumentId, e.AggregateId, x => x.ProtocolNumber = e.ProtocolNumber);

This small change on the projection allows us to do a little trick, the first time you look for object with a specific value, we create another in memory structure to immediately identify an object given that property. After the first search the index is created in memory and it is maintained for the subsequent indexes.