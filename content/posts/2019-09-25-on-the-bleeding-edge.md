---
template: post
title: On the bleeding edge
slug: /posts/bleeding-edge
draft: true
date: 2019-09-25T22:16:22.378Z
description: 'I''m customarily at the "bloody edge", not here.'
category: technical
tags:
  - faunadb
  - netlify
  - graphql
---
Once in awhile you stumble onto an interesting technology that is not quite ready for prime time.  My experience with FaunaDB shows that it is still evolving but, hey, it's free!

## How I got here

I've worked with AWS Lambda functions before, and Netlify uses that for its Functions service. What I haven't done is configure a database in the cloud, though I have used them.  Most importantly to me, though, is 1) easy to set up; 2) free (for now).

## FaunaDB

I don't care if the database is SQL or NoSQL--I've worked with both. FaunaDB is a NoSQL database, based on JSON documents \*and\* supports GraphQL. I've done some extensive work with CouchDB in the past, though Fauna appears superficially to be more like AWS Dynamo.  

I figured that once I get the resolvers written, I'd be using GraphQL primarily and use Fauna's FQL language only occasionally.  When importing a schema into Fauna, it will generate some basid GraphQL operations for you, but for more copmplicated stuff, such as bulk inserts/updates and indexed queries, you need to use FQL.

## An example

(muation operation here)

## How to get started

Clone or download the netlify-fauna-example.  Now, i like to make things difficult for myself, and one of the things i do to that end is to run `npm update outdated` and `npm audit fix`. I can expect to encounter issues when I do this, but practice I usually resolve them in an hour or two. 

Not this time.  Deleting node_modules, package-lock.json, and even doing a forced clean of the cache before reinstalling everything didn't work. What I would up eventally doing was switching over to yarn. That worked after a bit of struggle. I'll post my package.json file at the end of this article.

## Connecting your Netlify hostd app to fauna

Netlify makes React application hosting a breeze.  This blog is written using one of Netlify's CMS templates with a few small tweaks so far.  Netlify's CMS takes time to master, which I haven't yet, but I am able to use their content managemet dashboard to write posts such as this one
