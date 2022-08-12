---
layout: post
title: Custom checks on Bitbucket pipelines with npm scripts (part I)
slug: custom-checks-on-bitbucket-pipelines-with-npm-scripts-part-i
date_published: 2019-02-12T18:11:30.000Z
date_updated: 2019-02-12T18:31:21.000Z
tags: Javascript, Methodologies, Old Posts
excerpt: Configuring checks to run at the CI pipelines, for whatever reason we may need. In this case we check that the app version has been bumped.
image: "/images/posts/custom-checks-bitbucket.jpeg"
---

_Disclaimer: I never got to write part II, still leaving this here for memorabilia. Original post [here](https://medium.com/@Dor3nz/custom-checks-on-bitbucket-pipelines-with-npm-scripts-part-i-4686d5e77880)._

Working in a team, with pull request features with code review, and some sort of automation with a CI to prevent the merge of failing tests or _non_-_linted_ code into your master branch. Sounds familiar?

But, why should we stop at tests, or _linting_? What I’m going to explain here is for Bitbucket pipelines, mainly because it is what I’ve been using recently. But I’m pretty sure the idea and core concepts behind it are applicable to other CIs.

### Our use case

We work on multi-platform development with Nativescript and Angular. If you ever worked on mobile you know versioning is some serious stuff, so we wanted to make sure we didn’t skip versions when adding new features to our application.

We use semantic versioning, so automating the version bump made no sense as there is no way you can automatically decide if you are bumping a major, minor or patch version. We concluded that the specific number for the upcoming version had to be decided by the developer presenting the feature on his or her pull request.

### Our solution

As you may have already guessed, our solution is to add a check on our Bitbucket repository pipelines, which is an integrated CI for Bitbucket, similar to CircleCI or TravisCI if you’ve ever worked with them on Github.

#### The setup

Our script is going to check that the incoming version — *the version set by the developer on its feature branch*- is valid, i.e. superior than the current one.

To be sure of that we need two references, the current version and the incoming version. We decided that the incoming version should be the one defined in our `package.json` at the root of our source code, and this and only this reference to the version code is what the developer must update.

We also needed a version reference to compare to when running our script, and we decided to put it in a `version-lock.json` file also at the root of our app. This file would only contain the current version, like so:

{ "version": "1.0.0" }

#### Implementation

A quick note: the key to make this work is that we would take advantage of pipelines again, after the feature branch was merged, to run a script that would update this number with the current one at `package.json`, thus making sure the cycle is completed and non-complying pull requests coming later on would be blocked in a reliable way.

Moving on to the fun stuff, first thing to do was defining the script in our `package.json` and creating the `js` file that contained the logic:

package.json:

    {
      "scripts": {
        "version-check": "node version-check.js"
      }
    }

version-check.js

    function versionCheck() {
      // Compare package.json version to version-lock.json
      // log() if success
      // throw() on error
    }

    try { versionCheck(); } catch(error) { throw(error); }

I can’t really dive into the implementation of the script, but the important thing to know is that \***\*a pipeline will fail when a command exits with a non-zero exit code\*\***, e.g. a `throw()`.

After the script was created and defined in our `package.json` we could move on and tell the pipelines when to run it:

    image: node:6.9.4

    pipelines:
      default:
        - step:
            name: Build, test and check version
            caches:
              - node
            script:
              - npm install
              - npm run -- ng test --watch false --single-run true
              - npm run version-check

As you can see we placed it after our tests, `npm run version-check` would run the script that we created before, and, if the dev forgot to bump the version the build would fail like this:
![](https://www.dor3nz.com/content/images/2019/02/image.png)The pipe failing because the version is not bumped.
This forces the developer to commit a new change with the bump and this way we are never forgetting to merge a new feature without updating the version. Ta-da!

### Last remarks

I somehow teased it before: after merging, a new script on the pipelines should run and update the version on the `version-lock.js`. But that includes making changes on the code, committing them and pushing them via ssh from the pipes. It sure is hella fun to do but it also will take a few more lines to explain so I’m going to leave this post as is, we will dive into that in the next part.

Thank you for reading if you’ve made it so far, and please, if you have anything to say just say something in the comments or say hi to me on [twitter](http://twitter.com/dor3nz).
