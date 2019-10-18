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

The original project used netlify_lambda to build, using webpack and babel. I ran into some issues, fixed them, then ran into them again later. Rather than put up with the frustration, I decide to forego neltify lambda and chose Netlify Dev to build and deploy from the command line. The drawback was that I didn't have a local server, but I could deploy candidates to Netlify and test them without checking into github or deploying to production. The advantage was that there were less moving parts, since webpack and babel were no longer needed.

 

The FaunaDB example used Netlify Functions, which are built on top of AWS Lambda. I decided to follow that template when working with FireBase. Authentication of the client was different, though. Both FaunaDB and Firebase used an well-known environment variable for authentication. In Fauna's case, it was a simple token, but for Firebase it was a path reference to a generated certificate file. That in turn was read by each Firebase instance automatically. 

# Things change

To make a Netlify Function in your app, what you do is create a functions folder, then add a JavaScript file that exports a handler function.  The handler function is then invoked via a URL.  It gets more complicated when there are dependencies. In that case, the best approach is to use npm init in the functions folder to allow node package manager a way to add dependencies for the functions in the functions folder.

In the original FaunaDB project I used as a template, it employed netlify-lamba-  to package up (via webpack and babel) the functions and their dependencies into an output folder as part of the build process. It isn't really necessary to use those tools, however, especially if running a later version of node (which can be set via the environment variable AWS_LAMBDA_NODE_VERSION).  Eventually, to simplify the build process, I used zip-it-and-ship-it to package up the functiond and their dependiences.  

What I then realized was that as each function was bundled up into a zip file, that zip file contained its own node_modules folder.  What's more, the node_modules folder was "customized" to contain only those dependencies explicitly required by each function.  

# Frustrations

Converting the Fauna project to a Firebase one wasn't too bad.  I still wanted a GraphQL API, though, so that meant removing the Fauna package and then adding both Firebase and apollo-lambda-server to the package.json in the functions folder (using yarn, not npm). I then wrote some simple functions that 1) verified the GraphQL querying worked; 2) that I could connect to my Firebase instance.

The first step worked without a hitch , but the second did not. No matter where I put my credentials file, the function could not find it.  At this stage, the deployment that occured on the Netlify server was a black box to me.  It was only until I looked at what zip-it-and-ship-it was doing  that I discovered that the .json file in the /function folder was not getting included anywhere in the function zip packages. I surmised a similar issue was occuring on the Netlify deployment.

# Clever, but not clever enough

It took some thinking, but I realized that if I added my .json file in a local project, then added it as a dependency to my lambda function, then it would be included in the node_modules folder.  At that point, I would have a path:  ./credentials/mycred.json.  Yay!

Well, didn't work. The Netlify folks have a support forum and will reply even to freeloaders like myself. It took a bit of back-and-forth (and maybe a little whining) to finally nail down the issue: the lambda function doesn't have access to the file system on the server! So no matter where the .json file was located, the GOOGLE_CREDENTIALS variable, which takes a path, was not going to be able to load the credentials file.

It was also pointed out to me that maybe this is the path I want to go down, anyway.  With each lambda function essentially an island unto itself, with nothing shared amongst them, then having a lot of dependencies in each function's node_modules folder was quite a bit of overhead. This would affect spin-up time when accessing data via GraphQL from the lambda function.  I saw the point.

# So, in the end

Lambda functions are not a solution for everything.  In my case, I just want a simple datastore with a GraphQL frontend, but without exposing the GraphQL to the browser (for security). Though lambdas are one way ot tackling that,  the other is to co-locate the  React server with the GraphQL server, and have the former communicate with the latter, with authentication via, say, an MD5 hash of a passphrase set via environment variable.
