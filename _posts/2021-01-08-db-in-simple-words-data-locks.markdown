---
layout: post
title: "Databases in simple words: Data Locks"
date: 2021-01-08 20:00:00 +0100
categories: databases
excerpt: >-
  When you work with data from multiple connections, you need to manage access to the data.
  One of the possible approaches is data locks. Let's check it.
---
{%
    include image-ref.html
    file='20210108/cover.jpg'
    authorUrl='https://unsplash.com/@matt__feeney'
    authorName='Matthew Feeney'
    siteUrl='https://unsplash.com/photos/Nwkh-n6l25w'
    siteName='Unsplash'
%}

When you work with data, usually you would like to have a possibility to read/modify data from multiple connections (so-called in concurrent mode). Apart from that, we would like to avoid data corruption and inconsistency.

To achieve what we need there is a number of approaches in the area of **Concurrency Control**. One of them is called **Database Locking**. Let’s check it. In simple words.

### Database Locking of Concurrency Control

Let's say we have a database (SQL, NoSQL... it doesn't really matters), and we have multiple connections that are trying to read data from or write to the database.

{%
    include image.html
    file='20210108/01-simple-lock.png'
    alt='Explicit lock'
%}

**Write 1** is updating a part of the data. Soon after **Write 1** has been started, **Write 2** came with a wish to update the same data. Also, both reads want to include that data in the response. Our database doesn't want to have any conflicts between the first and second write. Also, it would be nice to include the last version of the data in the read. So write acquires a lock for that part of data and blocks other clients from accessing it. That lock is called Exclusive Lock and blocks everything else. Next write and reads will wait in a queue till the first write will complete.

> **Exclusive (write) lock** prevents the resource from being shared. All clients who are trying to access the resource will wait for a lock to be released.

*Write* usually means insert, update and delete operations.

Once **Write 1** will be executed, **Read 1** gonna kick in. It would be not nice if somebody will modify part of the data accessed by reading. It can lead to inconsistency of data and corrupted results. So it is better if a read will acquire a lock as well. But reading doesn't need such exclusivity as writing. There is no harm if another read will access the same data. So Read 1 acquires a shared lock.

> **Shared (read) lock** prevents the resource from being modified. Other reads can share a lock and work in parallel but writes need to wait for a lock to be released.

Reads and writes could have different complexity and scale. Some may require to lock not only a row or document but the entire table, collection, or even a database. For example, in the scope of one transaction **Write 1** needs to modify data in a few places.

{%
    include image.html
    file='20210108/02-intent-lock.png'
    alt='Explicit lock'
%}

While **Write 1** will be working on acquiring the exclusive lock for the first part, **Write 2** can get a lock for the second. Eventually, it can lead to the deadlock. To prevent that situation **Write 1** can obtain an exclusive lock for the entire collection and modify all needed data. It is a very heavy lock and will block all reads from the collection. To avoid it and unblock at least reads, **Write 1** can obtain an intent for exclusive lock on a collection.

> **Intent Exclusive lock** prevents any nested resource from being an exclusively locked. Other writes cannot acquire exclusive lock for any nested data.

Lover level of a lock, lighter it is. Higher level, more clients are waiting in a queue and lock is heavier.

Having locks in place ensures all ACID principles. It is safe to work with data in that way. But it is hard to provide a good performance and scale of data. Locks can be implemented in different databases a bit differently. You also can meet other, like update lock and intent to read.

### Where DB locks are used

Database locking is used for concurrency control in many databases. The typical example is MySQL. It is used in InnoDB to lock data on the [row level](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-shared-exclusive-locks), [index record](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-record-locks), and others. If you're inserting a new row with an auto-increment column it will [lock the entire table](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html#innodb-auto-inc-locks).

PostgreSQL implemented the possibility to use [Multi Version and Locking Concurrency Control](https://www.postgresql.org/docs/9.1/mvcc-intro.html). Similarly as in InnoDB, [table and row level locks](https://www.postgresql.org/docs/9.1/explicit-locking.html) are possible.

In Neo4j locking is used [on the nodes and relationships](https://neo4j.com/developer/kb/shared-vs-exclusive-transaction-locks/).

The locking mechanism is not too good from a performance perspective. But it provides the highest transaction isolation level which could be necessary for domains like financial.
