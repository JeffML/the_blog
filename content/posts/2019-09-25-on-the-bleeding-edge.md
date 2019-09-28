---
template: post
title: My take on FaunaDB and their GraphQL support
slug: /posts/bleeding-edge
draft: true
date: 2019-09-25T22:16:22.378Z
description: >-
  FaunaDB has functional support for GraphQL-based APIs. There's still some work
  to do.
category: technical
tags:
  - faunadb
  - netlify
  - graphql
---
I have one or two projects I maintain on Netlify, in addition to hosting my blog there. It's really an easy platform to deploy to, and has features like a content management system (CMS) and support for lambda functions (by way of AWS).

What I need for my latest project, though, was a database. The only one that's currently integrated into Netlify is FaunaDB, a NoSQL, document-oriented database. It also recently boasted support for GraphQL, which was a big plus. So I dove into it, at no charge.

## FaunaDB

Fauna has a unique approach to managing transaction isolation across globally distributed data stores. That's a mouthful, but basically it ensures that database records don't get out of sync when they're updated from far and wide.

## The application

I'm a chess player of middling ability and I want to set up data to do analysis of master-level chess games. SQL or NoSQL didn't matter to me--I've worked with both and either would support my modest needs.

I love GraphQL, but I don't want my schema exposed on the client side. The way to get around this is to have lambda functions to do the GraphQL requests, and have the client use those functions. 

## The implementation

I started with netlify-fauna-example*. This doesn't use GraphQL; instead the Netlify (lambda) Functions use FQL, Fauna's query language. To use GraphQL, I would first need to create a database on Fauna, and then import a GraphQL schema.

When I did this, a set of collections (akin to tables in SQL) were created based on my type definitions. Interestingly, new types were also added to handle stuff like managing relations between types. Some of this I'm sure was driven by Fauna-defined directives, such as @embedded and @relation...more on that in a bit.

### queries and mutations

There are certain queries and mutations that will be automatically implemented for you by Fauna _if_ you define them in the schema you created. So, if I define a type to hold chess opening information, called Opening, then I may include the following Query and Mutation definitions:

<code>

Fauna will implement the resolvers to these operations, so you don't have to write any resolver code.

### lambda functions

The original lambdas in netlify-fauna-example speak FQL. To convert these to GraphQL requests, just use a fetch library such as node-fetch, and make HTTPS requests to the GraphQL endpoint supplied by Fauna.

<code>

## Are we done now?  No.

Fauna's GraphQL support is in the functional, but formative, stage. One of the things I wanted to do was have batch insert ability so I wouldn't have to insert once opening at a time into the Opening collection. This isn't created by Fauna automatically, so I had to define a resolver for it. 

Fauna has a @resolver directive that can be used on mutation definitions that directs Fauna to use a user-defined function. For a simple collection like Opening, that's pretty simple:

<code> 

Things get trickier when you use directives such as @relation or @embedded. Remember I mentioned that Fauna created additional types once you're uploaded a GraphQL schema? **There's also collections backing those types.** Knowing what they are and how they work are important if you want the mutation stored properly.

Fauna generated input types as well, which seems like a bonus, but as a user you can't use them.  The way schema import works is that it will validate _your_ schema, not the one Fauna generates.  If you didn't define the input type, your schema won't validate. Unfortunately, if you do define the input type and give it the same name as the one Fauna would generate for you, you "override" the type that would have been generated. Fauna recommends that in this case, you should name your input types in a way that doesn't override the generated one. 

## Humble opinion follows...

Fauna's support team and community forum members have been exceedingly helpful with questions and even offer help with implementation. They're forthcoming when onsite documentation is incomplete or wrong. 

Probably the biggest hurdle to face is that in order to write resolvers, you will need to know FQL. I'll be candid and say that I am not a fan of FQL: it's low-level, verbose, repetitive, and nested, though it may appeal to functional programming mavens. It is possible to create functions and, with a deeper knowledge, one could probably build up a substantial library of helper methods to scale back the complexity.

I also thought the database slow:  bulk insert of a 1000 small records executed matters of seconds.

Ultimately, though, I want more control, even if it means a more work writing every resolver and defining every database structure. Normally I am pretty lazy, but too much magic can be too much.


----

\* My life isn't interesting enough, so when I clone a project like netlify-fauna-example, I will run `npm update outdated` and `npm audit fix`. I can expect to encounter issues when I do this, but in practice I usually resolve them in an hour or two. 

Not this time.  I deleted node_modules, package-lock.json, and even did a forced clean of the cache before reinstalling everything. Didn't work. I eventually switched over to yarn, deleted the above (but left the updated version info in package.json alone) and installed. After a few hiccups, success!

