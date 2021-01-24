---
published: true
layout: post
title: "Databases in simple words: Multi Version Concurrency Control"
date: 2021-01-24 20:00:00 +0100
categories: databases
excerpt: >-
  On the big data load it is important to process request as fast as possible. For databases it is a synonym
  to "do not lock requests". With that in mind a lot of engines implemented a Multi Version Concurrency Control.
---

{%
    include image-ref.html
    file='20210124/cover.jpg'
    authorUrl='https://unsplash.com/@bergerteam'
    authorName='Florian Berger'
    siteUrl='https://unsplash.com/photos/SzG0ncGBOeo'
    siteName='Unsplash'
%}

Serving many of concurrent requests, the database engine is responsible for managing conflicts between requests and keeping data integrity. The easy way to do so is to introduce data locks. Block requests from accessing the same data at the same time. In that case, usually reads are blocking writes. Writes are blocking everything.

To improve the performance would be nice to find a way not to block concurrent requests or at least block less. One of the approaches to achieve it is called Multi-Version Concurrency Control.

### How does it work

If all operation classify as read or write, lets list possible cases for blocking:

| --- |
| Read blocks Read |
| Read blocks Write |
| Write blocks Read |
| Write blocks Write |

Let's go through a list one by one.

**Read blocks read.** There is no danger to read the same parts of data in parallel. In locking, reads share a shared lock.

**Read blocks write**. If during a read data will be changed by another request, a result of reads may become corrupted. So we need to keep data unchanged from a read perspective. At the same time, we would like to keep the possibility to update the data. In other words, we want to:
 * keep the data unchanged from read perspective;
 * allow write to modify part of the data while others are reading it.

Right, we can make a copy of the data. As simple as that.

{%
    include image.html
    file='20210124/01-mvcc.png'
    alt='Multi-Version Concurrency Control'
%}

While reads are reading data from **version 1**, write is gonna create a copy of that data **version 2** and will update it without disturbing reads. Once a new version is ready, it will become a new version for further reads. That is where the name came from, **multi-version**. Multiple versions allow to have concurrent reads and writes without blocking.

**Write blocks read**. Obviously, the relation between reads and writes are symmetric. Multiple versions gonna help here as well.

**Write blocks write**. If write can create a new version of a data to modify it, probably multiple writes can create their own version of a data and try to modify it concurrently?

{%
    include image.html
    file='20210124/02-multiple-writes.png'
    alt='Concurrent Writes'
%}

If during commit of a **Write 2** updated data will conflict with a data from **Write 1**, we can cancel the change of **Write 2** and retry it from the beginning. That approach is called **optimistic locking**.

### Where MVCC is used

Even though Multi-Version Concurrency Control does not provide the highest isolation level, it enables a higher speed and performance which might be critical. That's why many databases are implementing that approach.

Multi-Version concurrency control is implemented in both, NoSQL and SQL. It is implemented in Wired Tiger which is used in [MongoDB](https://docs.mongodb.com/manual/core/wiredtiger/#snapshots-and-checkpoints). Also, it is used by default in [PostgreSQL](https://www.postgresql.org/docs/7.1/xact-read-committed.html).
