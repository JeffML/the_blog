---
template: post
title: >-
  MD5 vs SHA256 vs SHA1 - Which is the Most Secure Encryption Hash and How to
  Check Them
slug: /posts/hashes
draft: true
date: 2020-03-24T22:53:09.857Z
description: >-
  Hash functions are used for many things, including encryptions, validation,
  and indexing.
category: Technical
tags:
  - hash
  - encryption
  - security
---
# What's a hash function?

A hash function takes an input value (for instance, a string) and returns a fixed-length string value. An _ideal_ hash function has the following properties:

* it is instantaneously fast
* it has an infinitely large number of hash values it can return
* it generates a unique hash for every unique input
* it generates dissimilar hash values for similar input values

No ideal hash function exists, of course, but each aims to operate as close to the ideal as possible. Given that hash functions return fixed-length strings, the number of values that it can return is constrained to something less than infinite, yet a 256-byte hash has more possible values than there are atoms in the universe.

Ideally, a hash function returns no _collisions;_ that is to say, no two different inputs generate the same hash value.  For purposes of cryptography, hash collisions are a potential vector for decipherment. Also ideally, the distribution of hash values should be such that there is no observable pattern between input values and hash values.

Let's take the following two similar sentences:

"The quick brown **fox**."

"The quick brown **fax**."

Now compare the [MD5 hash values generated](https://www.md5hashgenerator.com/) from the two sentences above:

2e87284d245c2aae1c74fa4c50a74c77

c17b6e9b160cda0cf583e89ec7b7fc22

The two similar sentence generated very dissimilar hash values. This is a property useful both for validation and cryptography.



# Common hash functions

There are several hash function widely used. These functions were developed by mathematicians to help ensure that the functions come close to the ideal.  Over the course of further research, some have been shown to have weaknesses, though still good enough for noncritical use.

## \-- MD5



## \-- SHA256

## \-- SHA1

# Hash values for validation
