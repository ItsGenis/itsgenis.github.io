---
layout: post
title: Importing scss variables files without relative routes in Angular 6+
slug: importing-scss-variables-files-without-relative-routes-in-angular-6
date_published: 2019-02-18T18:31:49.000Z
date_updated: 2019-02-22T22:24:26.000Z
tags: Angular
excerpt: DRY and shiny variables scss files. All your tomatoes are belong to us.
image: "/images/posts/angular-scss-variables.jpeg"
---

Or how to style tomatoes across your app with one single variable.

I've been doing a lot of things around Angular lately, but none of them are styling. So today I had to style some stuff and, of course, I wanted to declare some reusable variables, you know, DRY and all, and found out that the `~` character [trick](https://netbasal.com/angular-cli-and-global-sass-variables-a1b92d8ca9b7) is [not](https://github.com/angular/angular-cli/issues/10077)[working](https://github.com/angular/angular-cli/issues/1253) in the latest versions of the framework CLI.

But there's a workaround.

So, imagine we have our nice  `/src/assets/styles/_colors.scss` file where we declare our shiny colors:

    $fruitColor: tomato;
    // some other less important colors

So, the way we do our shenanigans now is by adding this cute little `stylePreprocessorOptions.includePaths` in our `angular.json` file, inside the `build.options` property:

    "build":{
       "builder":"@angular-devkit/build-angular:browser",
       "options":{
          "stylePreprocessorOptions":{
             "includePaths":[
                "src/assets/styles"
             ]
          }
       }

And now we can do the following in any of our scss files:

    @import '_colors';

    .fruit.tomato {
      color: $fruitColor;
    }

And that's it. Happy tomato styling.
