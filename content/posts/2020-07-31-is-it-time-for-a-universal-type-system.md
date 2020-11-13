---
template: post
title: Is it practical to have a universal type system?
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

I recently made an attempt to help some open source project that took a GraphQL schema and generated viable SQL for various database dialects. The idea was that you'd write your GraphQL API, which includes type definitions, then generate functions that use dialect-specific DDL and DML to perform the underlying CRUD operations.  Similar to ORM or Swagger or countless other language-to-datastore mappings.

GraphQL has a simple yet extensible way of defining types in a declarative way. With support for directives, tweaks can be made to the GQL schema to inform a language generator (or validation engine) how to map GraphQL types to other language types. However, that binds the schema tightly to an SQL implementation, making it hard to later switch to something like another SQL database or even a NoSQL DB. On top of that, this tool was generating TypeScript, but what if the desired implementation language was Java or Python? Not everything has to run in a browser. 

## Deja vu all over again

Any sufficiently complex computer application has dealt with converting from one type system to another. I can tick off several:

* Object-Relational Mapping (ORM)
* UML CASE code generation
* Swagger
* Custom XML Schema to Java classes
* Semantic Object Model
* Castor
* Spring Roo

Most of these technologies aren't focused on type definitions, but they do play a big role, and some of these either support data mapping or form the basis of a larger data mapping framework. So it seems like types, type definitions, and data mapping taken together are a cross-cutting concern in many applications. Can the concept be abstracted away in some fashion?

## Primitive and compound data types

Types are definitions of data primitives and data structures. They are not objects with methods (okay, maybe getter and setter methods would be the exception). They are just data type definitions. There are two classes.

**Primitive Data Types**

* Integer
* Decimal
* Character
* Boolean

These are the bare-bones, and arguably not comprehensive enough. For instance a **string** of characters could also be considered a primitive type. Arrays are a primitive data type structure, and most languages support them. A more extensive primitive type set might include **date**, (including date/time) and **currency**, though these have complexities in presentation and range. Similarly with **float**, which could be argued is a formatted **decimal**, with storage considerations in the language implementation. 

Other considerations crop up: is **Integer** arbitrary in length? In the most abstract sense, no: it's just a mathematical concept. Yet in implementation it is bounded in some fashion, either by the implementing language(s) or by computer memory (e.g., BigInt in JavaScript). The same issue arises with **string** and any array: there is a practical maximum length, but in concept it's not of concern. More on this later.

**Compound Data Types**

These types are containers of primitive and other compound types. They go by various names:

* struct (C++)
* complex type (XML Schema)
* Plain Old Data Object (PODO, various languages)

All can be classified as "custom" types. The programmer defines them, gives the definition a name, and uses instances of them in code.

## Aspects of Universal Type System

A UTS schema does not define an implementation, it is purely conceptual. So an integer is just some whole number, and a string is a bunch of characters. It sketches out the kinds of data used in a domain. It brings to benefits: 1) as a concept, it doesn't get bogged down into implementation detail; 2) as a set of definitions, they can reasonably be represented in other languages. Those languages may introduce constraints, but that's another topic to address later.

From here on I will use the term "dialect" instead of "language", as well as "subdialect" to mean a variation of a language. I acknowledge the following to be "fuzzy", and that's fine. This article is intended as a thought piece and not a detailed outline with all issues worked out in or even accounted for.

<h3>Basic Components</h3>

The chief elements of a UTS schema are as follows:

<h4>The base definition</h4>

This is the core of UTS: a schema defining named types using primitives and composites. In my mind's eye it would look similar to a GraphQL typedef schema. As a graph-structured schema, it is also not unlike a relational database schema where table and columns follow a Table-per-Type structure. 

Type inheritance is not directly supported, because UTS is not conceived as a way to classify types in terms of is-a relationships. UTS would support type extension as a convenient shorthand to reduce redundancy in definitions, but extension would not imply is-a classification, though a language implementation could interpret it as such.

<h4>Domain extensions</h4>

A means of extending the UTS language would allow for things like the definitions of additional primitive types, or value ranges for types defined in the base definition. It may also define cardinality constraints between a type and it's child types. These are not language-specific constraints (though they incidentally could be), but constraints determined by the domain.

<h4>Dialect Extensions</h4>

These extensions determine the default implementations for data representation in a dialect, such as TypeScript or SQL. In addition, subdialect extensions would allow for refinement of dialects. For instance, a SQL dialect extension would have subdialects, each specific a particular RDBMS.

Essentially, each language would share a common set of types, but implement them according to the language constraints. Let's start with a basic diagram and build upon it further on.

<UTS with dialect extensions>

UTS base schema "sketches out" the types used in an application. It is useful to the data modeler modeling a business domain for the first time. There are several key element missing, though: **dialect defaults, dialect-to-dialect bindings,** and **custom configurations**.

**Dialect Defaults**

In order to generate types for a dialect, we have to know how to map a UTS type to dialect types.  For the majority of cases, reasonable default types can be determined for any dialect. That's not going to be sufficient in all cases, so there needs to be a way to override those default type mappings (see below).

**Dialect-to-Dialect Bindings**

So it's one thing to map UTS type definitions to actual language types, but in many scenarios one would want to cross language boundaries, say from JavaScript JSON "types" to SQL data definitions. Let's show that in the diagram:

<UTS diagram with binding definitions>

**Custom Configurations**

Like the UTS-to-Language mappings, bindings would have to allow for customization via configuration files. Those configuration files would modify the default bindings (as predefined by UTS for various languages). The configuration might change type-to-type bindings at various scopes: by language, by module, by type, language subdialect (say a DBMS SQL variant), or other scopes TBD.

<previous diagram, with configurations>

A universal type system really only makes sense if the default type-to-type mappings fall within the 80/20 rule: 80% of the defaults mappings are reasonable, and 20% need modification.  For primitive types, this would often be the case for a specific language; however, in a language-to-language binding, range out-of-bounds conditions could occur. 

<h4>Cross-dialect bindings</h4>

It's one thing to generate multiple language-specific data structure from a single UTS schema, but it is also often necessary to have two or more generated data structures in different languages map to one another. A binding specification defines those mappings, can it is assume a default binding can be generated by UTS.

The essential elements of a binding document would be a mapping between a UTS type (likely using some path specification as yet to be determined) and a meaningful declaration or set of directives that are specific to each dialect in the binding. Details of the dialect binding syntax would be specific to the implementation or language. 

**Example**

From a UTS schema, both a SQL schema and TypeScript PODO definitions are created. Both are fully conformant to the UTS base definition (plus any extensions). The TypeScript and database are to talk to each other, so that the TypeScript PODOs have mechanism to pull and push data from the SQL database. The TypeScript PODOs and SQL Tables (table-by-type) are said to be bound. This is similar to how ORMs work now.

<h3>Benefits</h3>

UTS would offer:

1.  rapid prototyping across different dialect implementations, useful for proof-of-concept demonstrations or for informing a base implementation. 
2. deliver an abstract schema that encapsulates domain concepts without bogging down in overly precise implementation details
3. reasonable cross-language type-to-type default mappings

Editors could be adopted to UTS to provide linkage to (and maintenance of) dialects, sub-dialects, and their configuration values. 

## How would it be implemented?

It would make sense to look at existing type definition languages and see if they could be adopted. I mentioned GraphQL before, and I think it is a candidate to base off of. The problem with current approaches to using GraphQL to generate code in other languages is that it invariably relies on directives to handle the find details of mapping. The issue is that directives tightly bind the GraphQL schema to a specific language implementation. 

Rather than use inline directives, the UTS proposed here would handle UTS-to-language and language-to-language bindings outside of the UTS schema itself.

**Should UTS be a graphical standard, like UML?**

Visual, diagrammatic approaches to schemas have not demonstrated improved productivity in development organizations (in my personal experience).  A large, complex schema diagram is just about has hard to understand and follow as a text-based schema with an editor that knows how to navigate it. Generating a visual representation of all or part of a UTS schema would in many cases be helpful, though, so that capability should exist.

A UTS engine might generate code or interpret schemas at runtime. In the latter case , schemas would be interpreted, compatible data instances created on-the-fly, and runtime validation of type-to-type mappings can be performed (perhaps even at a static code analysis level).  

## What are the problems?

The biggest issue with code generation has traditionally be performance and maintenance.  UTS would not offer a solution in that regard.  It's purpose it to get from concept to functional prototype in a rapidly, but at the same time provide a framework implementation that can be customized for performance and chosen technology platform. Reverse-engineering, versioning, and data migration issues are difficult problems that don't lend themselves to easy solutions, but they are solvable. Whether the complexity of the solutions lend themselves to a UTS-based framework would remain to be seen.

## My background
