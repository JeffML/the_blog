---
template: post
title: 'Living within your means: Firestore''s Spark Plan'
slug: /posts/livingfirestore
draft: true
date: 2019-11-29T18:47:27.760Z
description: >-
  With 50,000 reads and 20,000 writes, you might think your early development
  needs are covered.
category: Technical
tags:
  - firestore
  - cloud
  - database
---
So [I recently started a personal project](https://www.freecodecamp.org/news/netlify-functions-firebase-and-graphql-working-together-at-last/) where I wanted to use a database in the cloud. There are quite a few to choose from. My main criteria was that it be something low or no cost. Eventually I decided on [Firestore](https://firebase.google.com/docs/firestore), using the [Spark Plan](https://firebase.google.com/pricing).  This plan gives me 5Gb of storage, with 50K reads and 20K writes per day for free, which at the time seemed like plenty. I soon learned that a little carelessness can blow past those transaction limits pretty fast.

Firestore is a NoSQL document store database. Each NoSQL database is different and my a learning curve was steeper than expected. As you know, the best teacher is adversity, and I made my share of mistakes early on. One too many, though, and I'd hit the read or write limit of the plan, which sometimes could happen within and hour or two. Then it was time to call it a day.

Things are better now, so I offer these lessons learned:

# _Do_ start with the free plan

Yes, it is easy to overshoot the plan's limits, but those occurrences will force you to learn to be efficient with your reads and writes. You will become more cost-conscious of sequencing multiple database operations in an efficient way.

# Start with a small dataset

This may seem obvious, but by **small** I mean less than 100 documents total.  On my project, I first created a collection with 10K documents. I later realized that I made a mistake in the data I loaded, fixed that, found another, went to fix that, but... TRANSACTION_RESOURCE_LIMIT_EXCEEDED. Welp, done for the day.

# Take time in the design of your data model

Like the carpenter's axiom: "Measure twice, cut once.", you don't want to be adjusting your JSON document fields and structure piecemeal.  Oh, you _will_ of course, but you'll save yourself some transactions by practicing a little foresight. Write out a schematic of documents, their fields, and their relationships first. [Visualization is the key to happiness.](https://www.freecodecamp.org/news/inserting-uml-in-markdown-using-vscode/)

# Test and verify your data loading scripts 

You'll need scripts to populate the database from some other source.  That is the time to:

* verify what you loaded is what you expected
* correctly handle the no-data case for fields

I made mistakes in both cases.  First, when I loaded some string data into a document field, I hadn't immediately noticed that those strings had quotes already, so the stored strings had embedded quotes. It didn't seem that serious an issue, but it became a pain later when writing and testing searches on that field. Because there were a lot of documents, I spent a sizeable portion of my daily write quota to clean that up.

In the second case, I discovered that Firestore has no mechanism for determining the [existence of a property in a document](https://stackoverflow.com/questions/46806860/how-to-query-cloud-firestore-for-non-existing-keys-of-documents) (there is no _undefined_ check). There _is_ an [**exists** test for documents](https://firebase.google.com/docs/firestore/query-data/get-data#get_a_document), but not for document fields. The best practice is to populate missing data fields with **null**, and then do null equivalence tests in a where clause to find documents with "missing" properties.

# What a small dataset won't teach you

Once you worked out kinks on the small dataset, it is time to graduate to a larger one. With more documents to process, things like query efficiency, pagination and batch requests become important.

## Read in chunks, write in chunks

[Batch operations](https://firebase.google.com/docs/firestore/manage-data/transactions) allow for multiple read/writes on the database in a single transaction, meaning If any write operation fails, then all writes fail, and the database data retains its original state. Each operation in a batch count toward the total read/write quotas, so as such it doesn't help usage quotas. Also, when writing via batch operations, be aware there's a 500 operation limit per batch.

Be careful when correlating two documents (i.e., for every A document, there is an association by reference to a B document).  Don't fetch all of one first, then iterate through the other. That's a good way to chew up the transaction quota when debugging.

It is better to fetch a subset of the first collection, then iterate through it document by document. Associate these documents with document in the second collection by fetching **one** that matches criteria. Continue to do this until the entire first collection has been fetched.  When debugging, you can verify everything looks like it is working correctly and, if not, kill the process before a lot of transactions are run.

## How to limit query results

Firestore's query language isn't as richly expressive as SQL is, but there are still a number of ways to restrict your queries so that you don't overfetch data. Although technically there is no size limit for a POST response body, in practical terms there is.

Some mechanisms for limiting query results:

**where and compound where**

You can chain multiple where clauses together, similar to adding conditional expressions to a single where clause in SQL.

```
citiesRef.where('state', '==', 'CO').where('name', '==', 'Denver');
```

**limit and ranges**

You can limit the number of documents returned by a query by using chaining a limit clause at the end of the query object.

```
let biggest = citiesRef.where('population', '>', 2500000)
  .orderBy('population').limit(2);
```

You can also specify a range of records to retrieve via startAt/endAt or startBefore/endBefore constraints, which allows you to do cursor-based pagination.

```
let docRef = db.collection('cities').doc('SF');

return docRef.get().then(snapshot => {
  let startAtSnapshot = db.collection('cities')
    .orderBy('population')
    .startAt(snapshot);
  return startAtSnapshot.limit(10).get();
});
```

**in array query**

You can look for specific matches in an array.  This is good for enumerated values.

```
const usaOrJapan = citiesRef.where('country', 'in', ['USA', 'Japan']);
```

---

As demonstrated, it is possible to work within the limitations of the Spark Plan as you learn about Firestore. It's free, which is always a good place to start.
