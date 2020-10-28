---
template: post
title: Is it time for a universal type system?
slug: /posts/typesystem
draft: true
date: 2020-07-31T18:45:39.592Z
description: >-
  Historically,  each language defines its own types and type systems. What
  features would a universal type system have, and what benefits would it offer?
category: Technical
tags:
  - data types
  - GraphQL
  - OOP
  - classes
---
## Prologue

I was offering some help on an open source project which took a GraphQL schema and generated viable SQL for database dialects. The question that came up, though, was what makes GraphQL so special? GraphQL has a simple yet extensible way of defining types that makes it attractive as a basis for type definitions. It also has directives, which can be used to inform a language generator on how to map GraphQL types to other language types. But is GraphQL really the best choice as a basis of defining and application's types?

This question comes up again and again.  Any sufficiently complex computer application has to deal with translating one language's type system to another's. I can tick off several:

* Object-Relational Mapping (ORM)
* UML CASE code generation
* Swagger
* Custom XML Schema to Java classes
* Custom C++ to SQL types
* Castor
* Spring Roo

Type definitions and cross-language mapping was just a part of what these frameworks and technologies did, but it was a big part. Type definitions and mappings are a cross-cutting concern--so is there some meta type definition that can be useful in addressing this reoccurring need? Conceptually it seems possible, so let's explore the idea further.

## What are types?

Types are definitions of data primitives and data structures. They are not objects with methods, they're just data definitions. There are two classes:

**Primitive Data Types**

* Integer
* Decimal
* Character
* Boolean

That's the bare-bones. A **string** of characters could also be considered a primitive type. Arrays are a primitive data type structure, and most languages have them. Also worthy of consideration as primitives would be **date**, (including date/time) and curren**currency**, those these are 

****

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
