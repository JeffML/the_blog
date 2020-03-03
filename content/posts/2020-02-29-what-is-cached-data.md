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

In general terms, a cache (pronounced "cash") is a type of repository.  You can think of a repository as a depot of resources. In the military, this would be a supply depot, holding weapons, food, and other supplies needed to carry forward a mission.  

In computer science, these "supplies" are termed resources, where the resources are scripts, code, and document content. The latter is sometimes more specifically referred to as "assets";  these include text, static data, media, and hyperlinks. 

A cache's primary purpose to speed up retrieval of web page resources, speeding up page load times. Another critical aspect of a cache is to ensure that it contains relatively fresh data. This post will talk about two different ways to achieve this: memory caching and Content Delivery Networks (CDNs).



## Difference between data cache and data repository

As an aside, repositories (as in depot) also come to play in data architectures. These types of repositories are often designed to be fast, but that is not their primary purpose. Mostly they exist to hold vast amounts of data that can be accessed in a variety of ways.  For example, Amazon Glacier is a data repository that is designed to be cheap, but not fast. A SQL database, on the other hand, is designed to be flexible, fresh, and fast, but is seldom cheap and not as fast as a most caches (which are less flexible and not as fresh).



# Browser Cache: a memory cache

A memory cache stores data on the computer where the browser is running from. While the browser is active, cached data is stored in the computer's physical memory (RAM), but this data can later be written to the computer's hard drive for retrieval between browser sessions.

Because the cache is local to the computer, retrieval of data from the cache is much faster than retrieval from a remote internet server, many miles distant. The cache may store HTML, CSS, JavaScript, and media files.

As mentioned, speed of retrieval is the primary purpose of a cache, but it also must contain fresh data. In a browser, what constitutes "fresh" is determined by several criteria. First and primary, the browser looks a the URI of the resource being fetched and uses that to determine if it is in the cache.  So, the first time a user visits a website, at least some of the data on that site is unlikely to be in the cache, because the website's URI is new to the browser.  It may be that some content on the website has URIs that have been encountered before, so some of the site's data may be cached already.

Most of the time, the user is visiting a site that she or he has visited many times before. Websites have a propensity to change their content, though, and so whatever was cached on previous visits may not longer be fresh. How to determine when to refresh the cache by pulling new data from the server can be somewhat complex. 

## The direct method: cache-busting

Say I have a web page located at www.foobar.com/about.html which says everything about foobar.com that you would ever want to know.  Once you visit that page, all that data is cached by the browser.  Later, foobar.com is bought out by quxbaz.com, and the about page's content undergoes significant changes. The browsers cache doesn't have that new data, but it may thing what it has is current.  How might you convince the browser otherwise?

Since the browser relies on the URI to find items in the cache, if the URI changes then it's like the browser has never seen the data before, and will go fetch it if from the server. So, by changing the resource name from www.foobar.com/about.html to www.foobar.com/about2.html (or to www.quxbaz.com/about.html), the browser will see that new page URI is not in the cache, and immediately go fetch it from it's remote location.

You don't have to change the page name, though. Since the URI also includes a query string by definition, you can add a version parameter to the URI:  www.foobar.com/about.html?v=2hef9eb1.  In this case, the version parameter v is set new a new generated hash value whenever the content changes, or is triggered by some other process, such as a server restart. The browser sees that the query string has changed, and because query strings can affect what will be returned, it will fetch new content from the server.

## Indirect method:  header tags

Every resource request come with some meta information known as the header.  Conversely, every response also has header information associated with it. In some cases, the browser sees the response header values, and changes corresponding values in subsequent request headers. Some of those header values affect how caching is performed.

**Cache-Control**

When responding to a request, the server can send header tags to the browser indicating what behavior is should adapt when caching. If I load the page at https://en.wikipedia.org/wiki/Uniform_Resource_Identifier, the response contains this in its header record:

cache-control: private, s-maxage=0, max-age=0, must-revalidate

private means that only the browser should cache the document content

s-maxage and max-age are set to 0, which is equivalent to the alternate directive no-cache. s-maxage I'll get to later; it is max-age that is used by browsers.  You might think that this tells the browser not to cache the resource, but instead it merely indicates that the browser must revalidate the what it has in its cache, updating it only if necessary.  The re-validation is done through a HEAD request, which only returns header information about the resource being requested. This header information is then examined by the browser to determine if it must make a full GET or POST request to fetch a new version of the resource. If a refetch is not indicated, then the browser will use its cache copy.

must-revalidate commands the browser to revalidate the cached resource if it is stale. Since max-age is set to 0 in this case, the cached resource is immediately stale once received. The combination of max-age=0 and must-revalidate is equivalent to the directive no-cache.  Yes, there is a bit of redundancy in cache-control directives.

Cache-control directives are extensive.  A complete documented list can be found [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control).

**E-tag**

This is a token that the server sends and the browser retains until the next requests. This is only used when the browser knows that the resource's cache lifetime has expired.

**Other header tags affecting caching**

The header tags expires and last-modified are now obsolete, but still sent by most servers for backward compatibility with older browsers.  An example:

expires: Thu, 01 Jan 1970 00:00:00 GMT

last-modified: Sun, 01 Mar 2020 17:59:02 GMT

Here, the expires is set to the zeroth date (historically form UNIX operating system). That indicates that the resource expires immediately, just as max-age of 0 does.  Last-modified tell the browser when the latest update was made to the resource, which it can then use to decide if it should refetch it rather than use the cache value.

## What's a hard reload?

A hard reload forces the refetch of all resources on a page, be they content, scripts, stylesheets or media. Pretty much everything, right? Well, some resources are may not be explicitly included on a page. Instead, they can be fetched dynamically, usually after everything explicit has loaded.  The browser doesn't know ahead of time that this will happen, and when it does, the later requests (initiated by scripts, usually) will still use cached copies of those resources if available.

## What's clear cache and hard reload?

This operation clears the entire browser cache, which has the same effect as a hard reload, but additionally causes dynamically loaded resources to be refetched as well--after all, there's nothing in the cache, so there is no choice!



# CDN: a geo-located cache

 A CDN is a caches that stores data in geographically distributed locations so that round-trip times to retrieve data are reduced. This is achieved by having requests sent from a browser routed to a nearby CDN, thereby shortening the physical distance to where the requested data is stored. CDN's are also highly optimized to allow the fast servicing of a multitude of simultaneous requests.

A CDN is more like a resource repository than a browser cache is, but it's primary function is the same: return data quickly.
