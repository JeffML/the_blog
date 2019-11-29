---
template: post
title: Living within your means on the Firestore Spark plan
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
1. start with a small dataset
   1. design your data model
   2. build your data loaders
   3. verify what you loaded is what you expected
   4. The no-data case and null
      1. there is no exists query for fields
   5. what a small dataset won't teach you
2. larger datasets
   1. read in chunks, write in chunks
      1. when correlating firestore documents, pull only the data you need right away
      2. batch operations won't help reduce usage
   2. where and compound where
   3. limit
   4. startat/endat and startAfter/endAfter
   5. in array query
