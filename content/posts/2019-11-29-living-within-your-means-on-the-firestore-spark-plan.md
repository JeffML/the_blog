---
template: post
title: 'Living within your means: free tier storage plans'
slug: /posts/livingfirestore
draft: true
date: 2019-11-29T18:47:27.760Z
description: >-
  With 50,000 reads and 20,000 writes, you might think your development usages
  is covered. Think carefully.
category: Technical
tags:
  - firestore
---
So I recently started a project where I wanted to use a database in the cloud. There are quite a few, but I was looking for something low or no cost. I eventually decided on Firestore, using the Spark plan.  This gave me 50K reads and 20K writes for free, which at the time seemed like plenty. I quickly learned that a little carelessness can blow past those resource limits pretty fast.

I have experience with NoSQL databases, but each one is different, so a learning curve was involved. As with any learning experience, you learn mostly by making mistakes.  Too many mistakes, and you've hit the read or write limit of the plan...within and hour or two you could end up calling it a day.

So, without further ado, here's some hard-won knowledge I want to share: 

# Start with the free plan

Yes, it is easy to overshoot the plan's limits, but when you do so, you will be forced to learn to be efficient with your reads and writes. You will become more cost-conscious of database operations while the consequences of mistakes are not expensive.

# Start with a small dataset

This may seem obvious, but by **small** I mean less than 100 documents total.  I started with 10K documents, and when I made a mistake writing/updating those, that 20K write limit was quick to greet me.

# Take time in the design of your data model

Like the carpenter's axiom: "Measure twice, cut once.", you don't want to be adjusting your document fields and structure piecemeal.  Oh, it _will_ happen, but you can prevent some of it from happening with a bit of forethought.

# Write your data loaders and test them 

You'll need scripts to populate the database, and now is the time to:

1. verify what you loaded is what you expected
2. you correctly handle the no-data case and null

I made mistakes in both cases,  First, when I loaded some string data into my document fields, I hadn't noticed that those strings had quotes already. That meant my strings had embedded quotes, which was a pain for later searches. I eventually had to spend a sizeable portion of my write quota for the day cleaning that up.

In the second case, I discovered that Firestore has no mechanism for determining the existence of a document field.  There is an **exists** test for documents, but no corollary for document fields.  If there is no data for a field, the best practice is to populate it with null--you can do a null test in your queries.

# What a small dataset won't teach you



1. larger datasets
   1. read in chunks, write in chunks
      1. when correlating firestore documents, pull only the data you need right away
      2. batch operations won't help reduce usage
   2. where and compound where
   3. limit
   4. startat/endat and startAfter/endAfter
   5. in array query
