---
template: post
title: 'Netlify Functions, Firebase, and GraphQL: strung together at last!'
slug: /posts/netlify-firebase
draft: true
date: 2019-11-05T01:39:17.922Z
description: >-
  It wasn't easy, but I finally have a Netlify hosted React app talking to a
  lambda GraphQL server fronting a Firebase database. Whew!
category: Technical
tags:
  - Firebase
  - Netlify
  - lambda
  - GraphQL
---
In a \[previous post] I confessed defeat in my attempt to get a GraphQL server, running inside a Netlify lambda function, to talk to a Firestore instance.  The roadblock was that the firebase.js package required a environment variable with a path to the Firebase credentials file.  That was used by the Firebase class instance to automatically read and then load the file at that path. Problem is, once the application is up and running, I was told no file system access allowed. 

A short time later I found \[this technique], which allowed the credentials file to be passed in as a dependency. This works in Netlify Functions, because dependencies are bundled up with the lambda function, so that they are available to be loaded at runtime like any other dependency.  It's a subtle distinction: dependency loading vs file reading, but it worked, so I moved on.

# But...

It turns out there are more to dependencies in Netlify Functions than are readily apparent. In fact, there are several ways to deploy a lambda function to Netlify:

## zip-it-and-ship-it

This is a utility that works a lot like webpack:  it creates an archive file with your functions an all its dependencies.  Like webpack, it only pulls in dependencies that are actually used.  Here's an example:

\[example of source]

To use this in your build process, you create a build step in JavaScript, then call it in your package.json to producde an output folder. The netlify.toml configutation would be set up to indicate that lambda functions are in the output folder:

\[netlify.toml]

 



continuous deployment

## Netlify Dev
