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

Fauna has a @resolver directive that can be used on mutation definitions. It will direct Fauna to use a user-defined function. For a collection of simple types like Opening, the resolver FQL is pretty straightforward. 

First, I go to the FaunaDB Console Shell, and create the function `add_openings`:

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

Things get trickier when you use @relation or @embedded directives, though I have to say there is ample experience and help available on both the Community Forum and in the chat assistant when in the documentation. Let's take a look at updating a relation:

Here's the relation in my schema.  Note the directive:

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

In order to store a Game, I need to have also a Header (Opening is not required). So I have two collections that are related by a generated ref attribute like the one seen earlier for Opening. 

So I need to ensure that when I create a function for my addGames resolver to call, there has to be a Header created first. 

GraphQL Schema resolver attribute:

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

I'm not an FQL expert (see acknowledgments), but I can tell that this code (from innermost outward):

1. creates a header instance
2. selects its generated reference field "ref"
3. merges that reference as field "header" into the a data object "X"
4. "X" represents one element of the input array parameter "games" (GameInput)

### A note about input types

Mutation parameters require input types. In the previous mutations used  OpeningInput, HeaderInput and GameInput. I need to declare those in the GraphQL schema, otherwise when I import it via GraphQL Playground, it will fail validation.

Input types for opening and game could be something like:

```
input OpeningInput {
  fen: String!
  SCID: String!
  desc: String!
}

input HeaderInput {
  Event: String
  Date: String!
  White: String!
  WhiteElo: String
  Black: String!
  BlackElo: String
  ECO: String
  Result: String
}

input GameInput {
   header: HeaderInput! 
   fens: [String!]!
   opening: OpeningInput 
 }
```

Here's the problem: these types override the Fauna-generated input types, because Fauna uses these names, too. It's recommended instead to choose non-conflicting names for input types, such as MyOpeningInput or MyGameInput so there's no risk of conflict with the input types used by the generated mutations (createGame and createOpening):

```
  createOpening(data: OpeningInput!): Opening!
  createGame(data: GameInput!): Game!
```

The sad fact is, you can't use these generated types in your own schema, because they are create _after_ the import of your schema, which needs to pass validation checks to be imported.

## Where to go from here...

Fauna's support team and community forum members have been exceedingly helpful with questions and have even offered help with implementing mutations. They're also forthcoming when onsite documentation is incomplete or wrong. 

Performance wasn't great: the bulk insert of a 1000 small documents executed in matters of seconds, which is slow, but I suspect that the free version is not running in an optimized setup.

Probably the biggest hurdle to face is that in order to write custom resolvers, it is necessary to master FQL. I consider FQL a low-level, rather verbose query  language. It is possible to create functions, and one could probably build up a substantial library of helper methods to scale back the complexity, but it's also "nesty". If you look back at the add_games FQL, you can see that with just two relations the nesting is quite deep. If you wanted to have a mutation that created multiple related Collection items in one go, it could be a nesting nightmare. I encourage you to read the documentation and decide for yourself.

In the end, FaunaDB's strength (transaction isolation), is not applicable to what I want to do. I tried it because Netlify integrates it and there is early-stage support for GraphQL.  Also, it was free. I will definitely check on it in six months time and see how it is progressing.

- - -

\* My life isn't interesting enough, so when I clone a project like netlify-fauna-example, I will run `npm update outdated` and `npm audit fix`. I can expect to encounter issues when I do this, but in practice I usually resolve them in an hour or two. 

Not this time.  I deleted node_modules, package-lock.json, and even did a forced clean of the cache before reinstalling everything. Didn't work. I eventually switched over to yarn, deleted the above (but left the updated version info in package.json alone) and installed. After a few hiccups, success!
