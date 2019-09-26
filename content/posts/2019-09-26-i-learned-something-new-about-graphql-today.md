---
template: post
title: I learned something new about GraphQL today
slug: /posts/graphql-nullable
draft: false
date: 2019-09-26T17:08:25.056Z
description: >-
  The syntax for defining array types in GraphQL is not what you may think it
  is.
category: technical
tags:
  - graphql
---
I've been working with GraphQL for three years.  I thought I knew what the following definition meant:

```
type FooInput {
    field: [ValueInput!]!
}

type ValueInput {...}

mutation {
    addFoo(foo: FooInput) : Foo
}
```

But I was surprised when, in a mutation, GraphQL Playground allowed me to do this:

`addFoo(foos: {field: []})`

I had always though that `[ValueInput!]!` meant a non-empty array was required. Yesterday, someone pointed out to me that I was wrong.

So what's the difference between `[ValueInput!]!` and `[ValueInput]!`, then?  A brief Googling turned up this StackOverflow post: <https://stackoverflow.com/questions/46770501/graphql-non-nullable-array-list>

The answer by Daniel Reardon included this great table:

```
declaration accepts: | null | []   | [null] | [{foo: 'BAR'}]
-------------------------------------------------------------
[Vote!]!             | no   | yes  | no     | yes
[Vote]!              | no   | yes  | yes    | yes
[Vote!]              | yes  | yes  | no     | yes
[Vote]               | yes  | yes  | yes    | yes
```


Aha! It's all about what is nullable! Whereas `[ValueInput]!` allows both `[]` and `[null]`, `[ValueInput!]!` only allows `[]`.  The extra exclamation mark ensures that if a non-empty array is passed as a parameter, it cannot contain nulls, but it doesn't require any array elements.

You can't know everything.
