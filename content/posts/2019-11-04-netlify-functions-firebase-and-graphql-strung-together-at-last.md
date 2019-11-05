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

## netlify-lambda

## continuous deployment

## Netlify Dev

This is Netlifys CLI tool.  There are two main deployment options:  netlify deploy, which deploys to a Netlify server; and netlify dev, which deploys locally.  It requires a build step before deployment. 

Of the four, I find Netlify Dev the easiest to understand and use.  Note that if you use netlify function:create for your lambdas, the resulting folder structure will not be compatible with continuous deployment. Choose your poison wisely.

# The Project

First, you need to set up a Firebase account. Once done, you can download a credentials file in JSON format. I've attached a link to example source at the end of this post, but it will not run until you replace  the fake credentials file (fakecreds.json) with your real one.

## Creating the lambda function

I like Apollo's GraphQL libraries, but be aware that they are evolving rapidly.  For the lambda function, I used apollo-server-lambda to server up the GraphQL API.

The hangup I had previously was solved by using a different NPM package, firebase-admin, thus affirming the dictum: "There's always node module for it".  Finding the one that works can be a challenge, though.

With firebase-admin I can import the credentials file, rather than pass a useless path to it via environment variable as before with firebase.js. 

\[code snippet]

Now I create the lambda function using Netlify Dev (not \`netlify dev\`-- so confusing!): 

\`netlify create:function my-lambda\`

It will give me a bunch of prompts.  I want to use apollo-server-lambda, which is one of the prompts.

## Creating the client

Naturally I want to use apollo-client (in for a dime, in for a dollar as the saying goes). The easiest way I've found is to use create-react-app, and \[apollo-boost] to get a skeletal client up and running quickly.  Once this is done, I create a React component to trigger a call to my lambda, using the GraphQL API it provides.

\[code snippet]

And the response I'll show in a window alert box:

\[image]

## Added bonus!

Since I used apollo-server-lambda as a basis for the Netlify Function, I can go directly to the service endpoint via URL and it will bring up GraphQL Playground:

\[image]

Here I can test queries and mutations prior to embedding them in my React client code.

**Also**, when using \`netlify dev\` I have a hot server courtesy of create-react-app, so I can see result of code changes in "real" time. 

# Deployment

As I mentioned, there are several ways of deploying, but my preference is:

1. netlify dev for work-in-progress (local server)

2. yarn build to build the project; then netlify deploy to deploy it to a temporary netlify server

3. finally, netlify deploy --prod to deploy to my production server

----

That's it!  Here's a link to source.
