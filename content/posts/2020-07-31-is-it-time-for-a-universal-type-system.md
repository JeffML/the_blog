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
* Semantic Object Model
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

That's the bare-bones. A **string** of characters could also be considered a primitive type. Arrays are a primitive data type structure, and most languages have them. Also worthy of consideration as primitives would be **date**, (including date/time) and **currency**, though these have complexities in presentation and range.

Other considerations have to be made: is Integer arbitrary in length? Maybe it would make sense to add an **UnboundedInteger** type to handle Big Integer representations, and assume Integer is arbitrarily bounded depending on the language default. The **string** type also might be bounded, with an unbounded variant such as **text** that can be any length. Such issues have been tackled before by existing mapping technologies, so they should provide some reasonable guidance.

**Compound Data Types**

These types are containers of primitive and other compound types. They go by various names:

* struct (C++)
* complex type (XML Schema)
* Plain Old Data Object (PODO, various languages)

All can be classified as "custom" types. The programmer defines them, gives them a name, and uses instances of them in code.

## How might a Universal Type System work?

A UTS schema does not define an implementation, but can be used to generate implementations in other languages.Yes, it sounds like another type-to-type mapping framework, but UTS is language independent. Therefore, it describes data types used in a domain, but not in terms of how it is implemented in one more languages. It is language agnostic.

<h3>Basic Components</h3>

The chief elements of a UTS schema are as follows:

<h4>The base definition</h4>

This is the core of UTS: a schema defining named types using primitives and composites. It is much like any  hierarchical schema, and similar to relational database table and columns that follow a Table-per-Type structure. There would not support type inheritance: UTS is not designed to classify types in terms of is-a definitions. UTS would support type extension as a shorthand convenience to reduce redundancy in definitions, but extension does not imply is-a classification.

<h4>Domain extensions</h4>

A means of extending the UTS language would allow for things like the definitions of additional primitive types, or value ranges for types defined in the base definition. It may also define cardinality constraints between a type and it's child types. These are not language-specific constraints (though they incidentally could be), but constraints determined by the domain.

<h4>Language Extensions</h4>

These extensions determine the default implementations for data representation in a language, such as TypeScript or SQL databases. In addition, subdialect extensions would allow for refinement of dialects. For instance, a SQL dialect extension would have subdialects, each specific a particular RDBMS.

Essentially, each language would share a common set of types, but implement them according to the language constraints. Let's start with a basic diagram and build upon it as we dig deeper into the concepts.

<UTS with language extensions>

UTS base schema "sketches out" the types used in an application. It is useful to the data modeler modeling a business domain. There are several key element missing, though: **language defaults, language-to-language bindings,** and **custom configurations**.

**Language Defaults**

In order to generate types for a language, we have to know how to map a UTS type to a language types.  For the majority of cases, reasonable default types can be determined for any language. That's not going to work  in all cases, however, so there has to be a way to override those default type mappings, which I'll discuss later.

**Language-to-Language Bindings**

So it's one thing to map UTS types to actual language types, but in many scenarios one would want to cross language boundaries, say from JavaScript JSON "types" to SQL data definitions. Let's show that in the diagram:

<UTS diagram with binding definitions>

**Custom Configurations**

Like the UTS-to-Language mappings, bindings would have to allow for customization via configuration files. Those configuration files would modify the default bindings (as predefined by UTS for various languages). The configuration might change type-to-type bindings at various scopes: by language, by module, by type, language subdialect (say a DBMS SQL variant), or other scopes TBD.

<previous diagram, with configurations>

A universal type system really only makes sense if the default type-to-type mappings fall within the 80/20 rule: 80% of the defaults mappings are reasonable, and 20% need modification.  For primitive types, this would often be the case for a specific language; however, in a language-to-language binding, range out-of-bounds conditions could occur. 

<h4>Cross-language bindings</h4>

It's one thing to generate multiple language-specific data structure from a single UTS schema, but it is also often necessary to have two or more generated data structures in different languages map to one another. A binding specification defines those mappings, can it is assume a default binding can be generated by UTS.

The essential elements of a binding document would be a mapping between a UTS type (using some path specification as yet to be determined) and a meaningful declaration or set of directives that are specific to each dialect in the binding. Details of the dialect binding syntax would be specific to the implementation or language. 

**Example**

From a UTS schema, both a SQL schema and TypeScript PODO definitions are created. Both are fully conformant to the UTS base definition (plus any extensions). The TypeScript and database are to talk to each other, so that the TypeScript PODOs have mechanism to pull and push data from the SQL database. The TypeScript PODOs and SQL Tables (table-by-type) are said to be bound. This is similar to how ORMs work now.

<h3>Benefits</h3>

At the very least, UTS would offer rapid prototyping across different language implementations, or for proof-of-concept demonstrations. At a higher level, it can define types used in a domain without getting drawn into precise details too early in the process: just go with the defaults, and adjust later when they don't work.

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
