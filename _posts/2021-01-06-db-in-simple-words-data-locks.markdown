---
layout: post
title: "Databases in simple words: Data Locks"
date: 2021-01-06 20:00:00 +0100
categories: databases
excerpt: >-
  When you work with data from multiple connections you need to manage access to the data.
  One of the possible approaches is data locks. Let's check what is it.
---

When you work with data, usually you would like to have possibility to read/modify data from  multiple connections (so called in concurrent mode). Apart of that we would like to avoid data corruption and inconsistency.

To achieve what we need there is a number of approaches in the area **Concurrency Control**. One of them called **Database Locking**. Let's check what is it. In simple words.

### Database Locking in Concurrency Control

Let's say we have a database (SQL, noSQL... it doesn't really matters), and we have multiple connections which are trying to read data from or write to the database.

{%
    include image.html
    file='20210108/01-simple-lock.svg'
    alt='Explicit lock'
%}

**Write 1** is updating a part of data. Soon after **Write 1** has been started, **Write 2** came with a wish to update the same data. Also both reads wants to include that data in the response. Our database doesn't want to have any conflicts between first and second write. Also it would be nice to include the last version of the data in the read. So write acquires a lock for that part of data and blocks other clients from accessing it. That lock called Exclusive Lock and blocks everything else. Next write and reads will wait in a queue till the first write will complete.

> **Exclusive (write) lock** prevents the resource from being shared. All clients who is trying to access the resource will wait for lock to be released.

By *write* usually means insert, update and delete operations.

Once **Write 1** will be executed, **Read 1** gonna kick in. It would be not nice if somebody will modify part of data accessed by read. It can lead to inconsistency of data and corrupted result. So it is better if read will acquire a lock as well. But read doesn't need such exclusivity as write. There is no harm if another read will access the same data. So Read 1 acquires a shared lock.

> **Shared (read) lock** prevents the resource from being modified. Other reads can share a lock and work in parallel, but writes need to wait for lock to be released.

Reads and writes could have different complexity and scale. Some may require to lock not only row or document, but entire table, collection or even database. For example, in scope of one transaction **Write 1** needs to modify data in a few places.

{%
    include image.html
    file='20210108/02-intent-lock.svg'
    alt='Explicit lock'
%}

During acquiring the exclusive lock for first part, **Write 2** can get lock for second part. Eventually it can lead to a dead lock. To prevent that situation **Write 1** can obtain exclusive lock for entire collection and modify all needed data. It is very heavy lock, and will block all reads from collection. To avoid it and unblock reads, **Write 1** can obtain an intent for exclusive lock on a collection.

> **Intent Exclusive lock** prevents any nested resource from being exclusively locked. Other writes cannot acquire exclusive lock for any nested data.

Lover level of a lock, lighter it is. Higher level, heavier lock.

Having locks in place ensures all ACID principles. It is safe to work with such data. But it is hard to provide a good performance and scale of data. Locks can be implemented in different databases a bit differently. You also can meet others, like update lock and intent to read.
