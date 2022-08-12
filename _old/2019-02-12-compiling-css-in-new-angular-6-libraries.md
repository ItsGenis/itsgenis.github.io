---
layout: post
title: Compiling css in new Angular 6 libraries
slug: compiling-css-in-new-angular-6-libraries
date_published: 2019-02-12T18:18:58.000Z
date_updated: 2019-02-12T19:02:07.000Z
tags: Angular
excerpt: Compiling CSS in Angular libraries is not yet supported. But we may have a workaround.
image: "/images/posts/angular-compiling-css.jpeg"
---

_Disclaimer: This post was originally posted on [Medium](https://medium.com/@Dor3nz/compiling-css-in-new-angular-6-libraries-26f80274d8e5)._

Not so long ago Angular 6 [was released](https://blog.angular.io/version-6-of-angular-now-available-cc56b0efa7a4), and with it we got the [library support](https://github.com/angular/angular-cli/wiki/stories-create-library)for the angular-cli.

If you are anything like me, you probably started using it right out of the box. Problem is, right after I did, I got stuck with a small issue that is not yet supported in [ng-packagr](https://github.com/dherges/ng-packagr) (which is what Angular uses for the library system):

\***\*Compiling global styles.\*\***

Right now the ng-packagr is able to compile any styles file that is directly linked to a component in its `@Component` decorator via the`stylesUrl`property, but what If we want a global (think of it like a theme styles) to be imported in an optional way in our project when we install the library?

So after digging a little bit and [asking for some help](https://github.com/dherges/ng-packagr/issues/839) in the `ng-packagr` repo here is where I got.

### Let’s compile the css ourselves

Go on and create a new library in your new shiny Angular 6 project:

`ng generate library my-shiny-library`

Now navigate to your new shiny library directory inside the project:

`cd your-project/projects/my-shiny-library`

Now, we are going to need to \***\*install some devDependencies\*\*** in our library in order to bundle the css:

- [ng-packagr](https://github.com/dherges/ng-packagr)
- [scss-bundle](https://www.npmjs.com/package/scss-bundle)
- [ts-node](https://www.npmjs.com/package/ts-node)

scss-bundle is the tool to actually bundle the css and ts-node is a tool we will use to run a Typescript script to run the bundler that we will see in a moment.

**\***Note**\*\***: Don’t forget to add a _`_.gitignore*`* file inside the library directory to ignore _`_/node_modules*`* in your version control.\*

Now I like to keep things organised, so go on and create a scripts directory within the library project and create a `css-bundle.ts` (kudos to [Alan Agius](https://github.com/alan-agius4)for the script) file inside it:

    import { relative } from 'path';
    import { Bundler } from 'scss-bundle';
    import { writeFile } from 'fs-extra';

    /** Bundles all SCSS files into a single file */
    async function bundleScss() {
      const { found, bundledContent, imports } = await new Bundler()
        .Bundle('./src/_theme.scss', ['./src/**/*.scss']);

      if (imports) {
        const cwd = process.cwd();

        const filesNotFound = imports
          .filter(x => !x.found)
          .map(x => relative(cwd, x.filePath));

        if (filesNotFound.length) {
          console.error(`SCSS imports failed \n\n${filesNotFound.join('\n - ')}\n`);
          throw new Error('One or more SCSS imports failed');
        }
      }

      if (found) {
        await writeFile('./dist/_theme.scss', bundledContent);
      }
    }

    bundleScss();

In order for this script to work, we need a `_theme.scss` inside the `/src`directory of our library that actually \***\*contains and imports all the css that we want to bundle.\*\***

Now we can define a postbuild npm script to run the `css-bundle.ts`, inside the library’s `package.json` let’s define the build and postbuild scripts:

      "scripts": {
        "build": "ng-packagr -p package.json",
        "postbuild": "ts-node ./scripts/css-bundle.ts"
      },

So basically the build script `ng-packagr -p package.json` is the script that ng-packagr runs to build the library for distribution, and the postbuild script just runs the script that we set up above, the latter will be called just after the former thanks to the built-in post hooks from npm scripts.

If we now run `npm run build` from the root of the library we will get the build in the `/dist` folder with the compiled css.

\***\*The only drawback\*\*** from this approach is that we cannot take advantage of the angular-cli command `ng build --prod <library-name>` since we are forced to build from the library’s directory, furthermore, we also need to \****manually copy the *dist\* folder\*\*** into the Angular project’s `dist/<library-name>` folder.

### Using the styles in the Angular project

Now when you install the library in the Angular project (or copy it like we’ve seen above in dev), when you call whatever component you have in there it will not be styled, to import the styles into the project you will have to do so in the new `angular.json` file that we got in Angular 6:

                "styles": [
                  "src/styles.scss",
                  "dist/my-shiny-library/_theme.scss"
                ],

**\***Note**\*\***: Here we are using *`*dist*`* but if you have installed as a npm package remember to use *`*node_modules*`* ;)\*

### Last words

Once more, thank you to Alan Agius to [aid me](https://github.com/dherges/ng-packagr/issues/839) on this, and let’s hope that the new versions of the angular-cli bring some tooling to deal with this in the future.

As always, feel free to join the discussion in the comments or ping me in [Twitter](https://twitter.com/dor3nz?lang=en).

Cheers.
