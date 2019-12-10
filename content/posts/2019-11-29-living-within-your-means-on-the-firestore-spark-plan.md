---
template: post
title: 'Living within your means: Firestore''s Spark Plan'
slug: /posts/livingfirestore
draft: true
date: 2019-11-29T18:47:27.760Z
description: >-
  With 50,000 reads and 20,000 writes, you might think your early development
  needs are covered.
category: Technical
tags:
  - firestore
---
So I recently started a personal project where I wanted to use a database in the cloud. There are quite a few to choose from; my criteria  something low or no cost. I eventually decided on Firestore, using the Spark plan.  This gave me 50K reads and 20K writes for free, which at the time seemed like plenty. I quickly learned that a little carelessness can blow past those resource limits pretty fast.

Each NoSQL database is a little different and my a learning curve was steeper than expected. And, as you all know, we mostly learn by making mistakes. Too many mistakes, and you've hit the read or write limit of the plan: when I stared out, I could do that within and hour or two. Then I'd have to call it a day.

But learn I do, and so I will share some hard-won lessons.

# _Do_ start with the free plan

Yes, it is easy to overshoot the plan's limits, but those occurrences will force you to learn to be efficient with your reads and writes. You will become more cost-conscious of sequencing multiple database operations in an  efficient way.

# Start with a small dataset

This may seem obvious, but by **small** I mean less than 100 documents total.  On my project, I first created a collection with 10K documents. I later realized that I made a mistake, fixed that, found another, went to fix that, but... TRANSACTION_RESOURCE_LIMIT_EXCEEDED. Welp, done for the day.

# Take time in the design of your data model

Like the carpenter's axiom: "Measure twice, cut once.", you don't want to be adjusting your JSON document fields and structure piecemeal.  Oh, you _will_ of course, but you'll save yourself some transactions by practicing a little foresight. Write out a schematic of documents, their fields, and their relationships first. Visualization is the key to happiness.

# Write your data loaders and test them 

You'll need scripts to populate the database from some other source.  Now is the time to:

* verify what you loaded is what you expected
* correctly handle the no-data case for fields

I made mistakes in both cases.  First, when I loaded some string data into my document fields, I hadn't noticed that those strings had quotes already, so I would up with strings that had embedded quotes. That became a pain when later writing and testing searches on that string. I eventually had to spend a sizeable portion of my daily write quota to clean that up.

In the second case, I discovered that Firestore has no mechanism for determining the existence of a document field (in other words, there is no undefined check, like NOT EXISTS for a field). There _is_ an **exists** test for documents, but not for document fields. The best practice is to populate missing data fields with **null**. Then you can do a null equivalence test in a where clause to find documents with "missing" fields.

# What a small dataset won't teach you

Once you worked out kinks on the small dataset, it is time to graduate to a larger one. With more documents to process, things like query efficiency, pagination and batch requests become important.

## Read in chunks, write in chunks

Batch operations allow for multiple read/write operations on the database. A batch update is a transaction, meaning If any write operation fails, then all writes fail, and the database data retains its original state. All operations in a batch count toward the total read/write quotas, so batched operations don't reduce usage. Also, when writing via batch operations, be aware there's a 500 operation limit per batch.

When correlating two documents (i.e., for every A document, there is an association by reference to a B document), don't do what I did: fetch all the B's first, then attach references to A's that match some criteria.  If you have a lot of B's, then programming mistakes you later make in linking the A's to the B's could exhaust your read limit quickly. Instead, try to order the A and B documents in a way that you only have to fetch **n** number from each document collection at a time, and repeat as until all documents are read.

## How to limit query results

Firestore's query language isn't as expressive as, say,  SQL is. There are still a number of ways to restrict your queries so that you don't overfetch data. Remember that responses to POST requests have limits as well!

**where and compound where**

You can chain multiple where clauses together, similar to adding conditional expressions to a single where clause in SQL.

**limit and ranges**

You can limit the number of documents returned by a query by using chaining a limit clause at the end of the query object.

You can also specify a range of records to retrieve via startAt/endAt or startBefore/endBefore constraints, which allows you to do cursor-based pagination.

**in array query**

You can look for specific matches in an array.  This is good for enumerated values.

# In the end

You can work within the Spark Plan while learning about Firestore. It's a good place to start. <link to firestore article>.
