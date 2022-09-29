---
layout: post
title: Structuring routes and its namespaces with ExpressJS
slug: routing-namespaces-with-expressjs
date_published: 2019-02-20T17:54:07.000Z
date_updated: 2019-02-22T22:21:51.000Z
tags: Javascript
excerpt: A maintainable approach to structure routes in ExpressJS.
image: "/images/posts/expressjs-routing.jpeg"
---

_Note: This post assumes a little bit of knowledge on ExpressJS. You may be able to follow along anyways but I haven't made the effort._

---

I haven't played much with NodeJS in the past but this coming weeks I will be developing an API with ExpressJS, so exciting times ahead.

At work we already have some NodeJS APIs on production, but going over its source code there are some things that just feel wrong, one of them being the way they structure the routing.

The existing APIs have a single routing file per module, with every route declaration and its logic. For instance, the endpoints where you ask for a `user` collection and where you _create_ a `profile` are declared in the same file, and its inner logic after them. Also any other CRUD action related to this two resources, or any other resource related to the same module, are located in this file. As you can imagine, this goes thousands of lines down.

So basically what I did is:

- Separate routes by namespaces
- Adopt a route/controller pattern

This is the directory/files structure I aimed for:
![](https://www.dor3nz.com/content/images/2019/02/image-2.png)
Here's how I went about the implementation. First of all, we instantiate the module routes at the top level (in our case it's in an `app.js`, I've seen people using `server.js` but at the end of the day it's the file used to bootstrap the app, referenced in the `www` file):

    var express = require("express");
    var app = express();
    var coreRoutes = require("./modules/core/core.routes");

    // Declare the /core namespace as an entry point for any route
    // related to the core module.
    app.use("/core", coreRoutes);

Then we declare the namespaces that belong to the core module, inside core.routes.js:

    var express = require("express");
    var app = express();
    var usersRoutes = require("./users/users.routes");
    var profilesRoutes = require("./profiles/profiles.routes");

    app.use("/users", usersRoutes);
    app.use("/profiles", profilesRoutes);

    module.exports = app;

And now we can declare the specific resource routes, e.g. for users at users.routes.js:

    var app = require("express");
    var router = app.Router();
    var usersController = require("./users.controller");

    router.get("/", usersController.index);
    router.get("/:id", usersController.show);

    module.exports = router;

As you can see we are referencing the `usersController` , so here's how I implemented it in users.controller.js:

    exports.index = (req, res) => {
      res.json({ respose: "hello, we are the user collection!" });
    };

    exports.show = (req, res) => {
      res.json({ response: "hello, I am user " + req.params.id });
    }

And now we can access our API at `core/users/` for the users collection or at `/core/users/1` to check user with id 1. We would repeat the same process for the profiles resource. Of course the real logic is quite more complicated and I might add services to the pattern if I see the need to.

Point is, now I have a structured directory, much more easy to maintain and understand.
