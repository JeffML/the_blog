---
template: post
title: My take on FaunaDB and their GraphQL support
slug: /posts/bleeding-edge
draft: false
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
![](/media/vincent-van-zalinge-whrwb43vh9e-unsplash.jpg "Photo by Vincent van Zalinge on Unsplash")

I have one or two projects I maintain on [Netlify](https://www.netlify.com/), in addition to hosting my blog there. It's really an easy platform to deploy to, and has features like a content management system (CMS) and support for lambda functions (by way of AWS).

What I needed for my latest project, though, was a database. The only one that's currently "integrated" into Netlify is [FaunaDB](https://fauna.com/): a NoSQL, document-oriented database. It recently [boasted support for GraphQL](https://fauna.com/blog/the-worlds-best-serverless-database-now-with-native-graphql), which I consider a big plus. At no charge and with a simplified setup, why not try it? 

## The database

Fauna has a unique approach to managing [transactions across globally distributed data stores](https://fauna.com/blog/consistency-without-clocks-faunadb-transaction-protocol). That's a mouthful, but basically it ensures that database records don't get out of sync when they're updated from far and wide. This is a problem for global enterprises with high transaction volumes, but not for me.

## The application

I'm a chess player of middling ability and I want to set up data to do analysis of master-level chess games. SQL or NoSQL didn't matter--I've worked with both and either would support my application's modest needs.

I love GraphQL, and have been using it since 2016. I don't want really want my GraphQL schema exposed on the client side, though. The way to get around this is to have lambda functions to do the GraphQL requests, and have the client use those functions as a sort of proxy.

## The implementation

I started with [netlify-fauna-example](https://github.com/netlify/netlify-faunadb-example)*. This doesn't use GraphQL; instead the example's [Netlify Functions](https://www.netlify.com/docs/functions/) use FQL: [Fauna Query Language](https://docs.fauna.com/fauna/current/api/fql/). You can execute queries via [fauna shell](https://github.com/fauna/fauna-shell), or by [using a NodeJS client module](https://github.com/fauna/faunadb-js). The following uses the client to insert a **todoItem** into the **todos** collection:

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

![](/media/screenshot-2019-09-29-at-11.27.05-am-edited.png)

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

There are certain queries and mutations that will be automatically implemented for you by Fauna _if_ you define them in the schema you created. When I defined the type to hold chess opening information, I _may_ then include the following Query and Mutation definitions in my schema:

```
type Query {
 allOpenings: [Opening]
}
```

And FaunaDB will provide an implementation.

### lambda functions

The original lambdas in [netlify-fauna-example](https://github.com/netlify/netlify-faunadb-example) speak FQL. To convert these to GraphQL requests, just use a fetch library such as node-fetch, and make HTTPS requests to the Fauna GraphQL endpoint using an client like the one included with [apollo-boost](https://levelup.gitconnected.com/giving-react-a-lift-with-apollo-boost-74c6ff32894d): 

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

The code above requests the FEN strings for all the openings in the Opening collection. 

## Are we done now?  No.

Fauna's GraphQL support is in a functional but still formative stage. One of the things I wanted to do was have batch insert ability so I wouldn't have to insert once opening at a time into the Opening collection. This mutation isn't created by Fauna automatically (though it is ticketed feature request), so I had to define a resolver for it. 

Fauna has a @resolver directive that can be used on mutation definitions. It will direct Fauna to use a user-defined function written in FQL; these can be written directly in the shell. For a collection of simple types like Opening, the resolver FQL is pretty straightforward. 

First, I go to the FaunaDB Console Shell, and create the function `add_openings`:

```
CreateFunction({
  name: "add_openings",
  body: Query(
    Lambda(
      ["openings"],
      Map(
        Var("openings"),
        Lambda("X", Create(Collection("Opening"), { data: Var("X") }))
      )
    )
  )
```

Openings is an array, and the Map method executes Create call on each element.  I then add a @resolver directive to my mutation definition in the schema I will import (referred to as a **custom resolver**):

```
type Mutation {
   addOpenings(openings: [OpeningInput]) : [Opening]! @resolver(name: "add_openings" paginated:false)
}
```

Now when that mutation is executed via the GraphQL client, `add_openings` will be called and it will insert all games passed in as a parameter to the mutation. From the GraphQL client it looks like this:

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

## Fauna GraphQL's chicken and egg problem

You'll notice in the mutation above that I refer to the OpeningInput type.  In order for me to import my schema into Fauna, that type has to be defined.  But... when I imported Opening, Fauna auto-generated that type for me.  When I define it in my schema later (for the mutation), I essentially override that type. Since that generated type is used in generated mutations (ie., createOpening, singular),  by overriding that type definition in my own schema I could possibly break one of the generated mutations. 

The solution suggested is to not override the OpeningInput type, but to rename my input type to something like MyOpeningInput. That ensures that my import schema validates, and doesn't mess with what the generated mutations expect.

The problem gets messier, though, when you use the [@relation](https://fauna.com/blog/getting-started-with-graphql-part-2-relations) directive. That directive generates types that are used to relate two other type instances. 

Here's the relation in my import schema.  Note the directive:

```
type Game {
  header: Header! @relation
  fens: [String!]!
  opening: Opening @relation
}

type Header {
    Event: String
    Date: String!
    White: String!
    WhiteElo: String
    Black: String!
    BlackElo: String
    ECO: String
    Result: String
}
```

In order to store a Game, I need to have also a required Header (Opening is not required). The relation is maintained by a Fauna-generated **ref** field on the Header. The way this is defined for the mutation is through the use of a GameHeaderRelation type that allows the creation of both Game and Header in a single mutation. Here are the relevant generated types:

```
input GameHeaderRelation {
  create: HeaderInput
  connect: ID
}

input GameInput {
  header: GameHeaderRelation
  fens: [String!]!
  opening: GameOpeningRelation
}

type Mutation {
  createGame(data: GameInput!): Game!
}
```

Now to add a game with the required header info, I can call the mutation like so, from within the Playground:

```
mutation CreateGameWithHeader {
    createGame(data: {
        fens: [],
        header: { 
           create: {
           date: "2004.10.16", 
           white: "Morozevich, Alexander", 
           ...} ) {
        _id
        fens
        header {
          data {
            date
            white
          }
        }
    }
}
```

However if I want to create a batch upload mutation of games, I don't have access to the generated **GameHeaderRelation** type, or the other input types. Without those defined, my import schema won't validate if I try to use them in my bulk mutation definition. To reiterate: bulk mutations are a ticketed feature request, so they should be available soon. Yet this type of issue will arise regarding any custom resolver's use of types.

I though for a minute that the solution would be to download the generate schema (from Playground), then modify it with my bulk mutations.  However, I am _overriding_ **__**the otherwise-generated types on import, which is not what I want to happen.

### The workaround: write the resolver FQL

As stated, I need to ensure that when I create a function for my addGames resolver to call, there has to be a Header created first for each game.

The GraphQL Schema resolver attribute calls the FQL add_games function:

```
addGames(games: [GameInput]) : [Game]! @resolver(name: "add_games", paginated: false)
```

And here's the function definition for add_games:

```
CreateFunction({
  name: "add_games",
  body: Query(
    Lambda(
      ["games"],
      Map(
        Var("games"),
        Lambda("X", [
          Create(Collection("Game"), {
            data: Merge(Var("X"), {
              header: Select(
                ["ref"],
                Create(Collection("Header"), {
                  data: Select(["header"], Var("X"))
                })
              )
            })
          })
        ])
      )
    )
  )
}
```

I'm not an FQL expert (see acknowledgments), but this code is readable (from innermost outward):

1. creates a header instance
2. selects its generated reference field "ref"
3. merges that reference as field "header" into the a data object "X"
4. "X" represents one element of the input array parameter "games" (GameInput)

I should note that one of Fauna's engineers stated that maintaining references _by hand_ is "tricky".  It requires  understanding of what is going on beneath the covers. The [@embedded](https://docs.fauna.com/fauna/current/api/graphql/directives/d_embedded) type of relation may be easier to implement in FQL if the relation is one-to-one, as in this case.

## Where to go from here...

Fauna's support team and community forum members have been exceedingly helpful with questions and have even offered help with implementing FQL functions. They're also forthcoming when onsite documentation is incomplete or wrong. 

Performance wasn't great: the bulk insert of a 1000 small documents executed in matters of seconds, which is slow. However I didn't use pagination in my resolvers, so that may make a significant difference. It is also possible that the GraphQL features are in a slower debuggable configuration.

To write custom resolvers, it is necessary to master FQL. I'll be honest and state that is not a query language that I've taken a liking to, but my experience is limited and your mileage may vary. It initially strikes me as verbose and "nesty", though for simple CRUD operations it is fine.

I chose to try Fauna not for what it can do, but for convenience. I may now go check out DynamoDB and see if it more to my liking.

**Acknowledgements**

I'd like to thank Summer Schrader, Chris Biscardi, and Leo Regnier for their patience and insight.

- - -

\* My life isn't interesting enough, so when I clone a project like netlify-fauna-example, I will run `npm update outdated` and `npm audit fix`. I can expect to encounter issues when I do this, but in practice I usually resolve them in an hour or two.

Not this time.  I deleted node_modules, package-lock.json, and even did a forced clean of the cache before reinstalling everything. Didn't work. I eventually switched over to yarn, deleted the above (but left the updated version info in package.json alone) and installed. After a few hiccups, success!  Here are the dependency versions I would up with:

```
  "dependencies": {
    "apollo-boost": "^0.4.4",
    "chess.js": "^0.10.2",
    "encoding": "^0.1.12",
    "faunadb": "^2.8.0",
    "graphql": "^14.5.7",
    "graphql-tag": "^2.10.1",
    "node-fetch": "^2.6.0",
    "react": "^16.9.0",
    "react-dom": "^16.9.0",
    "react-scripts": "^3.1.1"
  },
  "devDependencies": {
    "http-proxy-middleware": "^0.20.0",
    "markdown-magic": "^1.0.0",
    "netlify-lambda": "^1.6.3",
    "npm-run-all": "^4.1.5"
  },
```
