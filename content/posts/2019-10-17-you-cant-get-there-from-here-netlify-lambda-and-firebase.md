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
[Awhile back](https://www.freecodecamp.org/news/how-to-use-faunadb/) a was exploring [Netlify's support for FaunaDB](https://www.netlify.com/blog/2019/09/10/announcing-the-faunadb-add-on-for-netlify/), which is a NoSQL document-oriented database with some special features to [handle transactions across dispersed database servers](https://fauna.com/blog/consistency-without-clocks-faunadb-transaction-protocol). I decided to try it because it was a convenient choice, since there was [example code](https://github.com/netlify/netlify-faunadb-example) I could start with.  The example used lambda functions as a frontend to the database. 

![fauna with lambda](https://user-images.githubusercontent.com/532272/42067494-5c4c2b94-7afb-11e8-91b4-0bef66d85584.png)

I modified the lambda functions to talk to the FaunaDB GraphQL API. While [that worked](https://www.freecodecamp.org/news/how-to-use-faunadb/), in the end I felt Fauna's GraphQL support wasn't quite ripe yet, so I looked around for alternatives.

I settled on [Cloud Firestore](https://firebase.google.com/docs/firestore/rtdb-vs-firestore). I based this new project on the Fauna example, swapping out the FaunaDB module with [apollo-server-lambda](https://www.npmjs.com/package/apollo-server-lambda), so that I can write my own GraphQL API and resolvers. One of the refinements I had to make was to push all my Netlify Function dependencies down to the /functions folder in my project (separate from the /src folder that contains my React client). To do this, I ran `npm init` while inside the functions folder, moved dependencies from the top-level package.json to the new /functions/package.json, and then ran `yarn install` to pull the packages into a new node_modules folder. The result was this:

![](/media/screenshot-2019-10-18-at-1.06.47-pm.png "The functions folder")

I'll talk about the subfolders later; the main thing to notice is that there's yarn files plus package.json, a node_modules folder, a schema folder, and some .js files for testing.

The original project used netlify_lambda to build, using webpack and babel. I ran into some issues, fixed them, then ran into them again later. Rather than put up with the frustration, I decide to forego neltify lambda and chose Netlify Dev to build and deploy from the command line. The drawback was that I didn't have a local server, but I could deploy candidates to Netlify and test them without checking into github or deploying to production. The advantage was that there were less moving parts, since webpack and babel were no longer needed. When going this route, you probably want to the build settings for your functions and set the environment variable **AWS_LAMBDA_JS_RUNTIME** to
**nodejs10.x**

# Things are not always as they seem

I am more familiar with GraphQL clients and servers than I am with lambda functions in the cloud, so there were some naive assumptions I made about how things were deployed in Netlify. I thought my functions were more or less copied over and build scripts run, and all would be happy and my functions would be ready. Of course, this is not at all what happens.

At first I was using netlify_lambda, and it would use webpack to create a functions_build output file. My netlify.toml configuration had that as the **functions** location.

```
[build]
  functions = "functions-build"
  # This will be run the site build
  command = "yarn build"
  # This is the directory is publishing to netlify's CDN
  publish = "build"
```

When I switch to using Netlify Dev, I dispensed with the output folder and just deployed the "unbundled" functions source. It turns out that's not quite the end of the story.

# Authentication woes

In the FaunaDB project, authentication was through an environment variable whose value was a simple token. A similar mechanism is used by Firebase, but instead of a token, the variable value is a path to a credentials file. My functions create a Firebase instance, and that instance looks for the env variable to locate the credentials file.

It seems like no matter where I put that credentials file or what path I used, the Firebase client would fail to find it. In the course of my research I came across a mention of Netlify's zip-it-and-ship-it utility, which other with other problems people recommended for zipping up functions along with their dependencies.

I tried it, changing my build process to call a NodeJS script that zipped up my functions to a functions-dist folder (changing the netlify.toml config to no point to that instead of the functions source folder). Although it didn't immediately fix my issues with the credentials file, I noticed some things.

![](/media/screenshot-2019-10-18-at-1.58.30-pm.png "The result of calling zip-it-and-ship-it")

 I now realized was that as each lambda function .js file was bundled up into a zip file, which also contained its own node_modules folder.  What's more, the node_modules folder was "customized" to contain only those dependencies explicitly required by each function.  

## I'm clever, but not clever enough

It took some thinking, but I realized that if I added my .json file in a local project, then added it as a dependency to each lambda function, it would be pulled in the node_modules folder.  At that point, I would have a path:  *./creds/mycred.json*.  Yay!

It didn't work. When I examined the zip files, the credentials were there, but the Firebase client still couldn't get to them. I announced my utter failure on the Netlify support forum, saying that I was contemplating joining a commune to learn to weave hammocks.

# AHA!

Taking pity on me, Dennis from Netlify let me know that you can not actually access the file system from within a lambda function. So what I was attempting was impossible. He suggested importing it into the lambda .js (already did that, just to make sure the credentials got pulled into node_modules). However, it doesn't appear that the Firebase client allows you to pull in credentials that way. 

Dennis also hinted that perhaps this isn't really what I should be doing, anyway. And I see his point. The only reason I went this route was because I was following an example, but swapping out the faunadb package with apollo-server-lambda might just have added a lot more weight to the lambda functions, which would affect spin-up times for dormant lambdas.

# Ditching lambda

Lambda functions are not a solution for everything.  In my case, I just want a simple datastore with a GraphQL frontend, but without exposing the GraphQL to the browser (for security). I can achieve the same ends by having a Node process host both a React client and a GraphQL server. I'm almost certain I won't run into any file system access problems, and if so, I'll switch to another method of authentication.
