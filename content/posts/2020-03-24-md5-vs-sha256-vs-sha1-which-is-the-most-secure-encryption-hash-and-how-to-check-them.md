---
template: post
title: >-
  MD5 vs SHA-1 vs SHA-2 - Which is the Most Secure Encryption Hash and How to
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
* generated hash values have to discernable pattern in their distribution

No ideal hash function exists, of course, but each aims to operate as close to the ideal as possible. Given that hash functions return fixed-length strings, the number of values that it can return is constrained to something less than infinite, yet a 256-bit hash has more possible values than there are atoms in the universe.

Ideally, a hash function returns no _collisions;_ that is to say, no two different inputs generate the same hash value.  For purposes of cryptography, hash collisions are a potential vector for decipherment. Also ideally, the distribution of hash values should be such that there is no observable pattern between input values and hash values.

Let's take the following two similar sentences:

"The quick brown **fox**."

"The quick brown **fax**."

Now compare the [MD5 hash values generated](https://www.md5hashgenerator.com/) from the two sentences above:

2e87284d245c2aae1c74fa4c50a74c77

c17b6e9b160cda0cf583e89ec7b7fc22

The two similar sentence generated very dissimilar hash values. This is a property useful both for validation and cryptography. Also important is that given a very large number of input values, the generated hash values are well-spread among all possible hash values.



# Common hash functions

There are several hash function widely used. These functions were developed by mathematicians to help ensure that the functions come close to the ideal.  Over the course of further research, some have been shown to have weaknesses, though still good enough for noncritical use.

## MD5

The MD5 hash function produces a 128-bit hash value. It was designed for use in cryptography, but weaknesses were discovered over the course of time, so it is no longer recommended for that purpose. However, it is still used for database partitioning and computing checksums to validate files transfers.

## SHA-1

SHA stands for Secure Hash Algorithm.   The first version of the algorithm is SHA-1, and was later followed by SHA-2 (see below).

Whereas MD5 produces a 128-bit hash, SHA1 generates 160-bit hash (20 bytes). In hexadecimal format, it is 40 digits long. Originally designed for cryptology applications, it was soon found to have weaknesses. As of today, it is no longer considered to be any more resistant to attack than MD5 is. The National Institute of Standards and Technology (NIST) recommends using SHA-256 (see following).

## SHA-2

The second version of SHA, called SHA-2, has many variants.  Probably the one most commonly used is SHA-256, which has hash value lengths of 256-bits, or 64 hexadecimal digits. Although not quite perfect (research ongoing), it is considerably more secure than either MD5 or SHA-1.  It is also about 20-30% slower to calculate a SHA-256 hash than either MD5 or SHA-1.

## SHA-3

This hash method was developed in late 2015, and has not seen widespread use yet. Its algorithm is unrelated to the one used by its predecessor, SHA-2. SHA3-256 is a variant akin to SHA-256, and the former takes slightly longer to calculate than the later.

# Using Hash Values for Validation

A typical use of hash functions is to perform validation checks. A frequently encountered usage is the validation of compressed collections of files, such as .zip or .tar archive files. Given an archive and its expected hash value, you can perform your own hash calculation to validate that the archive you received is complete and uncorrupted.

For instance, I can generate an MD5 checksum for a tar file in Unix using the following piped commands:

tar cf - files | tee tarfile.tar | md5sum -

I can then post the generated checksum on my site, along with the file archive download link. The receiver, once they have downloaded the archive, can validate that it call came across correctly by running the following command:

echo '2e87284d245c2aae1c74fa4c50a74c77 tarfile.tar' | md5sum -c

where 2e87284d245c2aae1c74fa4c50a74c77 is the generated checksum that was posted. Successful execution of the above command will generate an OK status like this:

echo '2e87284d245c2aae1c74fa4c50a74c77 tarfile.tar' | md5sum -c

tarfile.tar: OK
