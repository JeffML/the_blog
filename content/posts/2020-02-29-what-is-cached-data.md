---
template: post
title: What is Cached Data?
slug: /posts/cache
draft: true
date: 2020-02-29T20:52:08.452Z
description: >-
  A cache is a type of repository. It's primary purpose is to provide faster
  access to all kinds of data.
category: Technical
tags:
  - Cache
---
# What's a cache?

In general terms, a cache (pronounced "cash") is a type of repository. A repository contains "stuff", and is often synonymous with the word "depot". In computer science, the term cache has a more specific meaning: a repository of data designed to speed up data retrieval. I'm going to explain two types: 1) in-memory cache; 2) geo-located cache.  Both of these are commonly used when accessing web pages on the internet.

The term "data" is used in a the broadest sense: it may mean:

* attributes and their values, 
* streaming video, 
* images, 
* the textual content of a web page.
* presentational information, such as font sizes and component sizes
* scripts, usually JavaScript, that allows for interaction with the user 

As an aside, repositories (as in depot) also come to play in data architectures. These types of repositories are often designed to be fast, but that is not their primary purpose. Mostly they exist to hold vast amounts of data that can be accessed in a variety of ways.  For example, Amazon Glacier is a data repository that is designed to be cheap, but not fast. A SQL database, on the other hand, is designed to be flexible, fresh, and fast, but is seldom cheap and not as fast as a most caches (which are less flexible and not as fresh).

## The attributes of a cache

As mentioned, speed is of the essence in the design of a cache. That's not the only criteria, though: the cache's data has to be relatively fresh. Ensuring the freshness of a cache requires extra information about the data that is in the cache. 



Browser Cache: an in-memory cache

An in-memory cache stores data





Browser Cache

  A browsers employ an in-memory cache to avoid refetching of recent data

\- includes: HTML, CSS, Javascript, Media files

  It used the computer's physical memory for fast retrieval



CDN

  A CDN is a caches that stores data in geographically distributed locations so that round-trip times to retieve data are reduced. This is achieved by having requests sent from a browser routed to a nearby CDN, thereby shortening the physical distance to where the requested data is stored. CDN's are also highly optimized to allow the fast servicing of a multitude of simultaneous requests.





\* The essential elements of a cache

identity

expiration

updates



\* Problems with caches

stale data

\*\* clearing the cache (in-memory caches)

\*\* hard reload
