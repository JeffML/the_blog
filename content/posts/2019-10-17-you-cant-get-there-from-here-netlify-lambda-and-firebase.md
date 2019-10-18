---
template: post
title: 'You can''t get there from here: Netlify Lambda and Firebase'
slug: /posts/firebase
draft: true
date: 2019-10-17T22:25:46.582Z
description: When traipsing the bleeding edge you discover the undocumented things.
category: Technical
tags:
  - Netlify
  - Lambda
  - Firebase
---
[Awhile back](https://www.freecodecamp.org/news/how-to-use-faunadb/) a was exploring [Netlify's support for FaunaDB](https://www.netlify.com/blog/2019/09/10/announcing-the-faunadb-add-on-for-netlify/), which is a NoSQL document-oriented database with some special features to [handle transactions across dispersed database servers](https://fauna.com/blog/consistency-without-clocks-faunadb-transaction-protocol). I decided to try it because it was a convenient choice, since there was [example code](https://github.com/netlify/netlify-faunadb-example) I could start with.  The example used [lambda functions](https://docs.netlify.com/functions/overview/) as a frontend to the database. 

![fauna with lambda](https://user-images.githubusercontent.com/532272/42067494-5c4c2b94-7afb-11e8-91b4-0bef66d85584.png)

I modified the original lambda functions to talk to the FaunaDB GraphQL API (instead of [FQL](https://docs.fauna.com/fauna/current/api/fql/)). While [that worked](https://www.freecodecamp.org/news/how-to-use-faunadb/), in the end I felt Fauna's GraphQL support wasn't quite ripe yet, so I looked around for alternatives.

I settled on [Cloud Firestore](https://firebase.google.com/docs/firestore/rtdb-vs-firestore). I based this new project on the Fauna example, swapping out the **faunadb** module with [apollo-server-lambda](https://www.npmjs.com/package/apollo-server-lambda), so that I could write my own GraphQL API and resolvers. One of the refinements [I had to make](https://github.com/netlify/netlify-lambda/issues/112) was to push all my Netlify Function dependencies down to the /functions folder in my project (separate and at the same level as the /src folder that contains my React client). To do this, I ran `npm init` while inside the functions folder, moved a set of dependencies from the top-level package.json to the new /functions/package.json, added a [webpack.functions.js](https://github.com/netlify/netlify-lambda/issues/112#issuecomment-488644361), then ran `yarn install` to pull the packages into a new node_modules folder. The result was this:

![](/media/screenshot-2019-10-18-at-1.06.47-pm.png "The functions folder")

I'll talk about the subfolders later; the main thing to notice is that there's yarn files, plus package.json, a node_modules folder, a schema folder, and some .js files for testing.

The original project used [netlify_lambda](https://github.com/netlify/netlify-lambda) to build, which uses webpack and babel. I ran into [some issues](https://github.com/netlify/netlify-lambda/issues/183), fixed them, then ran into them again later. Frustrated, I decided to forego netlify-lambda and chose [Netlify Dev](https://www.netlify.com/products/dev/) to build and deploy from the command line. The drawback was that I didn't have the ability to launch a local server, but I could deploy candidates to Netlify and test them without first checking source into github or deploying directly to production. There were less moving parts, since webpack and babel were no longer needed. When going this route, you probably set the environment variable **AWS_LAMBDA_JS_RUNTIME** to
**nodejs10.x** in the _Build & deploy_ settings for your functions.

# Things are not always as they seem

I am more familiar with GraphQL clients and servers than I am with lambda functions in the cloud, so there were some naive assumptions I made about how things got deployed in Netlify. I thought functions were more or less copied over and build scripts run on the server, where all would be happy and my functions would be callable via URL. This is not at all what happens.

When I started with netlify_lambda, it would use webpack to create a functions_build output file. My netlify.toml configuration had that as the **functions** location.

```
[build]
  functions = "functions-build"
  # This will be run the site build
  command = "yarn build"
  # This is the directory is publishing to netlify's CDN
  publish = "build"
```

When I switch to using [Netlify Dev](https://www.netlify.com/products/dev/), I dispensed with the output folder and just deployed the "unbundled" /**functions** source. That's not the end of the story, though.

# Authentication woes

In the FaunaDB project, authentication was through an environment variable whose value was a simple token. A similar mechanism is used by Firebase, but instead of a token, the variable value is a path to a credentials file that you generate through the FireBase console. The lambda functions create a Firebase instance, and that instance looks for the env variable to locate the credentials file for authentication.

It seems like no matter where I put that credentials file or what path I used, the Firebase client would fail to find it. In the course of my research I came across a mention of Netlify's [zip-it-and-ship-it](https://github.com/netlify/zip-it-and-ship-it) utility, which other people with other problems recommended for bundling up functions in zip files.

I tried it, changing my build process to call a NodeJS script that zipped up my functions to a **functions-dist** folder (changing the **netlify.toml** config to no point to that instead of the **functions** source folder). Although it didn't immediately fix my issues with the credentials file, I noticed some things.

![](/media/screenshot-2019-10-18-at-1.58.30-pm.png "The result of calling zip-it-and-ship-it")

I began to realize that as each lambda function .js file was bundled up into a zip file, it also contained its own **node_modules** folder.  What's more, the node_modules folder was "customized" to contain only those dependencies explicitly required by each function.  

## Clever, but not clever enough

It took some thinking, but I decided that if I added my .json file in a local project, then made it a dependency to each lambda function, it would be pulled in the node_modules folder.  At that point, I would have a path:  _./creds/mycred.json_.  Yay!

It didn't quite work--when I examined the zip files, the credential files were there in each zip archive, but the Firebase client still couldn't get to them. I confessed my utter failure on the Netlify support forum, saying that I would join a commune to learn to weave [hammocks](https://www.twinoakshammocks.com/).

# Help!

I must have evoked some pity, Dennis from Netlify soon responded and let me know that lambda functions cannot actually access the file system. What I was attempting (loading credentials via a file path) was impossible. He suggested importing it into the lambda .js (which I had already done); yet it doesn't appear that the Firebase client allows you to pull in credentials via an import. 

That aside, Dennis sort of hinted that perhaps this isn't really what I should be doing, anyway.  He had a point. The only reason I went this route was because I was following an example, but swapping out the faunadb package with apollo-server-lambda _might_ just have added a lot more weight to the lambda functions, which possibly would affect spin-up times during [cold starts](https://hackernoon.com/im-afraid-you-re-thinking-about-aws-lambda-cold-starts-all-wrong-7d907f278a4f).

# Ditching lambda functions

Lambda functions are [not a solution for everything](https://hackernoon.com/developer-challenges-of-serverless-and-aws-lambda-8b8d5e299a34).  In my case, I only wanted a simple datastore with a GraphQL frontend, without exposing the GraphQL queries in the browser console. I can achieve the same ends by having a Node process host both a React client and a GraphQL server. I'm (almost) certain I won't run into any file system access problems, and if so, I'll switch to[ another method of authentication](https://cloud.google.com/docs/authentication/production).
