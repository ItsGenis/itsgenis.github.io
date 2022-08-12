---
layout: post
title: Running specs outside src in Angular
slug: running-specs-outside-src-in-angular
date_published: 2019-02-12T18:23:54.000Z
date_updated: 2019-02-12T18:32:30.000Z
tags: Angular
excerpt: Ever wanted to run the .spec files outside your /src folder? Probably not. Anyway here's how you would do it.
image: "/images/posts/specs-outside-src-angular.jpeg"
---

#### Motivation

I was a bit hesitant to write this one down because of its triviality, but since I had to dig a little myself to work it out, here it goes. May it serve anyone or even my future self.

By our current client requirements, we have some modules that are shared across different apps living inside a mono-repo. The apps are mainly their own bootstrapping where we insert the other modules. This modules live within a `modules/` directory, at the same level that the src folder is placed.

The way Angular CLI works, when running `ng test` in an app’s directory, Karma runs all of the test files ending with `spec.ts` that are inside the src directory. What I wanted to achieve was to also run the spec files inside the modules folder using the same command.

---

#### Implementation

Let’s start by having a look at the `ng test` configuration inside the `angular.json` :

    "test": {
      "builder": "@angular-devkit/build-angular:karma",
      "options": {
        "main": "src/test.ts",
        "polyfills": "src/polyfills.ts",
        "tsConfig": "src/tsconfig.spec.json",
        "karmaConfig": "src/karma.conf.js",
        "styles": [
          "src/styles.scss"
        ],
        "scripts": [],
        "assets": [
          "src/favicon.ico",
          "src/assets"
        ]
      }
    },

The bits that are relevant here are the `"main": "src/test.ts"` and `"tsConfig": "src/tsconfig.spec.json"`, where the testing environment is initialised and where the files to be compiled by the typescript compiler (and its options) are specified, respectively.

If we have a look at the `test.ts` file we can see how the testing environment is initialised and how, using Webpack’s `require.context()`, the test files are being loaded. It is possible to load extra files, in this case our modules context, following the same steps by adding this two lines at the end of the file:

    const moduleContext = require.context('./../modules', true, /\.spec\.ts$/);
    moduleContext.keys().map(moduleContext);

Finally, the last step is to specify our spec files for the typescript compiler inside `tsconfig.spec.json`:

    "include": [
      "**/*.spec.ts",
      "**/*.d.ts",
      "../modules/**/*.spec.ts"
    ]

Running `ng test` after this little changes will run the tests from inside src (as usual) plus the ones inside the modules folder.

---

This is all for today. Thank you for reading and feel free to reach me on [Twitter](https://twitter.com/dor3nz).
