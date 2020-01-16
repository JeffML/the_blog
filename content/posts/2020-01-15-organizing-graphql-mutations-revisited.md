---
template: post
title: 'Organizing GraphQL Mutations, Revisited'
slug: /posts/organizegraphql
draft: false
date: 2020-01-15T22:56:28.349Z
description: >-
  Unlike GraphQL Queries, Mutations require dependable order of execution. This
  post explains one way of achieving that.
category: Technical
tags:
  - GraphQL
---
# Background

Some time back [I wrote an article](https://www.freecodecamp.org/news/organizing-graphql-mutations-653306699f3d/) about how to organize GraphQL mutations in a heirarchy, much like how it is [sometimes done for queries](https://blog.hasura.io/graphql-and-tree-data-structures-with-postgres-on-hasura-dfa13c0d9b5f/#fetching-comments).  Awhile later, Anders Ringqvist informed me of a problem with my approach: it didn't guarantee that mutations will be executed in order.  This is important because, unlike queries, mutations modify state, and one mutation may require the completion of a previous mutation to operate correctly.  After verifying this, I wrote a [follow-up article](https://www.freecodecamp.org/news/beware-of-graphql-nested-mutations-9cdb84e062b5/) as a retraction of sorts.

# Never say never

Matthew Lanigan recently contacted me and suggested a very simple Promise-based mechanism for ensuring order-of-execution in "nested" mutations. It's quite elegant and seems to work without introducing any side-effects.

# The mechanism

Here's an example of a flat set of mutations that operate on a collection of books:

```
mutation {
  addBook(ISBN: "978-3-16-148410-0", title: "Schematics of the Illudium Q-36 Explosive Space Modulator", author: "Martian, Marvin") {
    references {
      title
    }
  },
  updateBook(ISBN: "978-3-16-148410-0", title: "Overview of the Illudium Q-36 Explosive Space Modulator") {
     success
  }
}
```

Here are those same operations rewritten to use a single Book entity:

```
mutation {
  Book(ISBN: "978-3-16-148410-0") {
    add(title: "Schematics of the Illudium Q-36 Explosive Space Modulator", author: "Martian, Marvin") {
      references {
        title
      }
    },
    update(title: "Overview of the Illudium Q-36 Explosive Space Modulator") {
     success
    }
}
```

The problem with the latter example is that I can't be sure that **add** will execute before **update**.  Since I can update a book that hasn't been added yet, that's a problem  

# The solution

Mathew's elegant and simple approach encapsulates mutation operations inside a parent class. That class, when constructed, creates a **promise** field that holds a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).  The Promise is immediately resolved, so that any subsequent **then** clause will be invoked immediately. 

# An example

```
class Sequential {  
   constructor() { this.promise = Promise.resolve() }
}
```

The Sequential class is then encapsulating mutation;  it doesn't actually mutate anything, it is just a container for mutation operations. Let's next add a mutation operation:

```
const msg = (id, wait) => new Promise(resolve => {
  setTimeout(() => {
    console.log({id, wait})
    let message = `response to message ${id}, wait is ${wait} seconds`;
    resolve(message);
  }, wait)
})

class Sequential {
  constructor() {
    this.promise = Promise.resolve()
  }

  message({id, wait}) {
    this.promise = this.promise.then(() => msg(id, wait))
    return this.promise
  }}
```

Notice that the **message** mutation waits for the current Promise resolve.  Since the Promise invoked in the constructor is resolved immediately,  the \`this.promise.then(...)\` statement will execute instantaneously _if it is the first mutation invoked_. If it is not the first mutation, then it will wait for the resolution of previous mutation. The code to follow will make this clear.

Note that the outer **msg()** function also returns a Promise. It is written to behave asynchronously, resolving only when the passed-in wait time has expired. This method of delaying execution will be handy during testing.

## The Schema

The GraphQL schema is pretty basic:

```
type Query {
  noop: String!
}

type MessageOps {
  message(id: ID!, wait: Int!): String!
}

type Mutation {
  Sequential: MessageOps
}
```

I can add as many operations to MessageOps as desired, with a corresponding implementation of those operations in the Sequential class as previously shown. The GraphQL server requires at least one Query definition, so the **noop** query, which does nothing, fulfills that obligation.

## The Resolver

The code for the resolvers is straightforward:

```
const resolvers = {
  Mutation: {
    Sequential: () => new Sequential(),
  }
}
```

## The execution

Now we are able to execute my mutation operations and see what happens.  There are two things to watch for: 1) the response from the mutation call, and 2) the console output. This latter is important, because only the console output will indicate the order in which the mutations were invoked and completed.

Here's the test:

```
mutation sequential {
  Sequential {
    message1: message(id: 1, wait: 3000)
    message1a: message(id: 11, wait: 2500)
  }
  Sequential {
    message2: message(id: 2, wait: 1000)
    message2a: message(id: 22, wait: 750)
  }
  Sequential {
    message3: message(id: 3, wait: 500)
    message3a: message(id: 33, wait: 250)
  }
  Sequential {
    message4: message(id: 4, wait: 100)
    message4a: message(id: 44, wait: 50)
  }
}
```

The response is as follows:

```
{
  "data": {
    "Sequential": {
      "message1": "response to message 1, wait is 3000 seconds",
      "message1a": "response to message 11, wait is 2500 seconds",
      "message2": "response to message 2, wait is 1000 seconds",
      "message2a": "response to message 22, wait is 750 seconds",
      "message3": "response to message 3, wait is 500 seconds",
      "message3a": "response to message 33, wait is 250 seconds",
      "message4": "response to message 4, wait is 100 seconds",
      "message4a": "response to message 44, wait is 50 seconds"
    }
  }
}
```

This looks like everything occurred in order, but that can be deceiving: The order of results in the returned JSON matches the order of the mutation calls, but that order can be different than that in which the mutations completed. It is the console output from each invocation of msg() that lists the actual order-of-execution:

```
{ id: '1', wait: 3000 }
{ id: '11', wait: 2500 }
{ id: '2', wait: 1000 }
{ id: '22', wait: 750 }
{ id: '3', wait: 500 }
{ id: '33', wait: 250 }
{ id: '4', wait: 100 }
{ id: '44', wait: 50 }
```

Thus we have proof that the mutations are actually occurring in the correct sequence: msg #1, which takes 3000 milliseconds, finishes execution well before msg #44, which only takes 50 milliseconds to finish. This is because each mutation operation is only invoked when the previous mutation has finished. Voila!

As I await to be informed of the next "gotcha", please feel free to examine the complete gihub project [here](https://github.com/JeffML/nested_mutations). It contains some extra goodies that you can run your own GraphQL mutations against.
