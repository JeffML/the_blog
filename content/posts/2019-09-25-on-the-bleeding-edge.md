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

What I needed for my latest project, though, was a database. The only one that's currently "integrated" into Netlify is FaunaDB: a NoSQL, document-oriented database. It recently boasted support for GraphQL, which I consider a big plus. At no charge, why not try it? 

## The database

Fauna has a unique approach to managing transaction isolation across globally distributed data stores. That's a mouthful, but basically it ensures that database records don't get out of sync when they're updated from far and wide. This wasn't of interest to me, but for big enterprise, high transaction volume applications it would be.

## The application

I'm a chess player of middling ability and I want to set up data to do analysis of master-level chess games. SQL or NoSQL didn't matter to me--I've worked with both and either would support my application's modest needs.

I love GraphQL, but I don't want my schema exposed on the client side. The way to get around this is to have lambda functions to do the GraphQL requests, and have the client use those functions as a sort of proxy.

## The implementation

I started with netlify-fauna-example*. This doesn't use GraphQL; instead the Netlify (lambda) Functions use FQL, Fauna's query language. Here's an example:

todos-create.js

```js
  ...
  /* construct the fauna query */
  return client.query(q.Create(q.Ref('classes/todos'), todoItem))
    .then((response) => {
      console.log('success', response)
      /* Success! return the response with statusCode 200 */
      return callback(null, {
        statusCode: 200,
        body: JSON.stringify(response)
      })
    }).catch((error) => {
      console.log('error', error)
      /* Error! return the error with statusCode 400 */
      return callback(null, {
        statusCode: 400,
        body: JSON.stringify(error)
      })
    })
```

To use GraphQL, I need to create a database on Fauna, and then import a GraphQL schema. Once you've created an account on Fauna, you can do all this through their dashboard.

![](/media/screenshot-2019-09-28-at-12.58.14-pm-edited.png "FaunaDB Console")

Once done, a set of collections (akin to tables in SQL) were created based on my imported GraphQL type definitions. Interestingly, new types and fields were also added to handle stuff like identifying instances and managing relations between types. For instance, my type for Opening was:

```gql
type Opening {
  desc: String!
  fen: String!
  SCID: String!
}
```

and when I go to the dashboard, open GraphQL Playground, and look at the schema, I see:

![](/media/screenshot-2019-09-28-at-1.05.59-pm.png "Fauna-generated types")


### queries and mutations

There are certain queries and mutations that will be automatically implemented for you by Fauna _if_ you define them in the schema you created. When I defined the type to hold chess opening information, I may then include the following Query and Mutation definitions in my schema:

```
type Query {
 allOpenings: [Opening]
}
```

### lambda functions

The original lambdas in netlify-fauna-example speak FQL. To convert these to GraphQL requests, just use a fetch library such as node-fetch, and make HTTPS requests to the GraphQL endpoint supplied by Fauna using an client like the one included with apollo-boost: 

```js
import ApolloClient from 'apollo-boost';
import gql from 'graphql-tag'
import fetch from 'node-fetch'
import authorization from './authorization'

const URL = 'https://graphql.fauna.com/graphql'

const client = new ApolloClient({
  uri: URL,
  fetch,
  request: operation => {
    operation.setContext({
      headers: {
        authorization
      },
    });
  },
})


exports.handler = (event, context, callback) => {
  const allOpeningFens = gql`    
  query openings {
      allOpenings {
        data {fen}
      }
    }
  `;


  client.query({ query: allOpeningFens })
    .then(results => {
      callback(null, {
        statusCode: 200,
        body: JSON.stringify(results),
      })
    })
    .catch(e => callback(e))
}
```
The code above requests the FEN strings for all the openings in the Opening collection. As mentioned, I didn't have to write a resolver for allOpenings, Fauna created it for me.

## Are we done now?  No.

Fauna's GraphQL support is in a functional but still formative stage. One of the things I wanted to do was have batch insert ability so I wouldn't have to insert once opening at a time into the Opening collection. This mutation isn't created by Fauna automatically, so I had to define a resolver for it. 

Fauna has a @resolver directive that can be used on mutation definitions. It will direct Fauna to use a user-defined function. For a collection of simple types like Opening, it's pretty straightforward. 

First, I go to the FaunaDB Console Shell, and create the function `add_openings2`:

```
CreateFunction({
  name: "add_openings",
  body: Query(
    Lambda(
      ["openings"],
      Map(
        Var("games"),
        Lambda("X", Create(Collection("Opening"), { data: Var("X") }))
      )
    )
  )
```

Then I add a @resolver directive to my mutation definition in the schema I will import:

` addOpenings(openings: [OpeningInput]) : [Opening]! @resolver(name: "add_openings" paginated:false)`

Now when that mutation is executed via the GraphQL client, `add_openings` will be called and it will insert all games passed in as a parameter to the mutation. From the client it looks like this:

```
import ApolloClient from 'apollo-boost';
import gql from 'graphql-tag'
import fetch from 'node-fetch'
import authorization from './authorization'

const URL = 'https://graphql.fauna.com/graphql'

const client = new ApolloClient({
  uri: URL,
  fetch,
  request: operation => {
    operation.setContext({
      headers: {
        authorization
      },
    });
  },
})


exports.handler = (event, context, callback) => {

  const addScidDocs = gql`
  mutation($scid: [OpeningInput]) {
    addOpenings(openings: $scid) {desc}
  }
  `

  const json = JSON.parse(event.body)

  client.mutate({
    mutation: addScidDocs,
    variables: { scid: json },
  })
    .then(results => {
      console.log({ results })
      callback(null, {
        statusCode: 200,
        body: JSON.stringify(results),
      })
    })
    .catch(e => callback(e.toString()))

  // callback(null, { statusCode: 200, body: event.body })
}
```

Things get trickier when you use directives such as @relation or @embedded. Remember I mentioned that Fauna created additional types once you're uploaded a GraphQL schema? **There's also collections backing those types.** Knowing what they are and how they work are important if you want the mutation stored properly.

Fauna generated input types as well, which seems like a bonus, but as a user you can't use them.  The way schema import works is that it will validate _your_ schema, not the one Fauna generates.  If you didn't define the input type, your schema won't validate. Unfortunately, if you do define the input type and give it the same name as the one Fauna would generate for you, you "override" the type that would have been generated. Fauna recommends that in this case, you should name your input types in a way that doesn't override the generated one. 

## Humble opinion follows...

Fauna's support team and community forum members have been exceedingly helpful with questions and even offer help with implementation. They're forthcoming when onsite documentation is incomplete or wrong. 

Probably the biggest hurdle to face is that in order to write resolvers, you will need to know FQL. I'll be candid and say that I am not a fan of FQL: it's low-level, verbose, repetitive, and nested, though it may appeal to functional programming mavens. It is possible to create functions and, with a deeper knowledge, one could probably build up a substantial library of helper methods to scale back the complexity.

I also thought the database slow:  bulk insert of a 1000 small records executed matters of seconds.

Ultimately, though, I want more control, even if it means a more work writing every resolver and defining every database structure. Normally I am pretty lazy, but too much magic can be too much.

- - -

\* My life isn't interesting enough, so when I clone a project like netlify-fauna-example, I will run `npm update outdated` and `npm audit fix`. I can expect to encounter issues when I do this, but in practice I usually resolve them in an hour or two. 

Not this time.  I deleted node_modules, package-lock.json, and even did a forced clean of the cache before reinstalling everything. Didn't work. I eventually switched over to yarn, deleted the above (but left the updated version info in package.json alone) and installed. After a few hiccups, success!


