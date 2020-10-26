---
template: post
title: Is it time for a universal type system?
slug: /posts/typesystem
draft: true
date: 2020-07-31T18:45:39.592Z
description: >-
  Historically, type systems have been tied to a specific language or
  implementation. Is it possible or desirable to have a type definition language
  that is independent of both?
category: Technical
tags:
  - data types
  - GraphQL
  - OOP
  - classes
---
## What is it?

* primitive data types
* compound data types\
  * containers, relations, and arrays

## Why do we need it?

* defining types is a cross-cutting concern
* however, most types systems are based on implementation or programming language\
  * this means they can't be shared across implementation boundaries\
  * hence, there's a lot of type-mapping involved (or no type validation at all)
* UTS would be the root definition for types across all boundaries\
  * it would not, in itself, be complete: dialects and subdialects would need to provide specific implementation details\
  * but, reasonable and configurable defaults are practical\
  * with code generation or introspection, a UTS based development approach would at the least provide rapid prototyping and proof-of-concept demonstrations, which are important when exploring new technological solutions and approaches to old problems.
* At the base level, UTS type definitions provide simplified documentation of types used in an architecture\
  * editors can be developed to provide linkage to (and maintenance of) dialects, sub-dialects, and their configuration values. Code generation tools would also be needed.

## What would it be based on?

The idea of a universal type definition language is unique, but has largely been implemented as visual schemas.  

* UML
* SOM
* ???

Visual, diagrammatic approached to type definition and code generation have not demonstrated improved productivity in development organizations.  A large, complex diagram is just about has hard to understand and follow as code. That's not to say they are useless, but beyond a certain level of complexity they don't seem to help.

## How would it be implemented?

One approach would be to adopt an existing type system and simplify or extend it to be a UTS. I like GraphQL's type definition language, for instance, and I can see it being a basis for the UTS, minus the operational aspects such as API service points (queries, mutations, and subscriptions) and type resolvers.  

### Directives and tight binding

TODO

## The Elements of UTS

* The base definition
* The base extensions\
  * additional primitives\
  * cross-cutting constraints, such as string length or cardinality ranges
* dialects\
  * default type implementations for a language (e.g., TypeScript) or data store (e.g. SQL DB)
* sub-dialects\
  * default type implementations for a specific implementation (e.g. Postgres, React-based forms)
* configurations\
  * these would be used to override default type implementations as create by code-generators
* cross-dialect bindings\
  * from a UTS definition I can generate, say, a SQL schema and also TypeScript classes, where both implementations are fully conformant to the base definition. However, I would want my TypeScript PODOs to pull and push data from the SQL database, therefore binding the two together. Assuming the SQL generated follows the paradigm of Table-per-Type (or Table-per-Type Hierarchy) then the mappings between PODO classes in TypeScript to table rows in the database should be straightforward, and this is basically how language-specific ORMs function now.\
  * the essential elements of a binding document would be a mapping between a UTS type (using some path specification as yet to be determined) and a meaningful declaration or set of directives that are specific to each dialect in the binding. Details of the dialect binding syntax would be specific to the implementation or language. 



## What are the problems?

The biggest issue with code generation has traditionally be performance and maintenance.  UTS would not offer a solution in that regard.  It's purpose it to get from concept to functional prototype in a rapidly, but at the same time provide a framework implementation that can be customized for performance and chosen technology platform. Reverse-engineering, versioning, and data migration issues are difficult problems that don't lend themselves to easy solutions, but they are solvable. Whether the complexity of the solutions lend themselves to a UTS-based framework would remain to be seen.

## My background
