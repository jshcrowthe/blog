---
title: "Setting up Polymer.js (0.5)"
slug: "/setting-up-polymer-js-in-express"
description: "This is the currently recommended way to manage your web components."
date: "2015-05-03T07:12:00.000Z"
published: true
categories: 
  - "Polymer"
  - "Web components"
  - "boilerplate"
---

## Bower
This is the currently recommended way to manage your web components. In harmony with that line of thinking I have set up a .bowerrc file that moves the `bower_components` directory to `public/components` (I am using an Express app and public is my static route).

This setup allows developers can build their own web components, use existing community components and maintain a file structure that is logical and easy to vulcanize. 

## Polyfill Inclusion
For browsers that don't have native Web Component support the file `webcomponents.js` contains all of the needed polyfills to make web components work in your browser. At Chrome Dev Summit 2014 (video [here](https://www.youtube.com/watch?v=kV0hgdMpH28)) there was a demo on Polymer performance and they shared the following code snippet. 

```javascript
  if ('registerElement' in document
    && 'createShadowRoot' in HTMLElement.prototype
    && 'import' in document.createElement('link')
    && 'content' in document.createElement('template')) {
    console.log("Using WebComponent Enabled Browser");
  } else {
    document.write("<script src="webcomponents.js"><\/script>");
  }
```

It's pretty self explanatory: Don't include the Polyfill unless you actually need it. Although we do use a `document.write` to write the script tag onto the screen (which can be considered gross) remember that, for those browsers that don't support `rel="import"` we need this on the page before the `<link>` tags for the rest of our app to even work!

*NOTE: Make sure that you include this polyfill **BEFORE** you include jQuery or any other library that "owns the DOM" (jQuery, D3, ang as you may run in to errors if you load the polyfill second.*

## Vulcanize
By using vulcanize to concatenate web components you can reduce the total number of calls the browser makes significantly! Rather than each web component being requested individually (along with its assets) you are able to concatenate Web Components, JS Files, and CSS files into one file that you then include in a `<link>` tag. As you can imagine for performance this is a **HUGE** win!

Below is my gulpfile.js that handles all the heavy lifting:

```javascript
var gulp = require('gulp');
var COMPONENTS_DIR = './public/components';
var COMPONENT_DEST = './public/html';
var vulcanize = require('gulp-vulcanize');
var minifyHTML = require('gulp-minify-html');
var minify_CSS_JS = require('gulp-minify-inline');
var rename = require('gulp-rename');

gulp.task('x-app', function() {
  return gulp.src(COMPONENTS_DIR + '/x-app/x-app.html')
            .pipe(vulcanize({
              inline:true,
              dest: COMPONENT_DEST
            }))
            .pipe(minifyHTML())
            .pipe(minify_CSS_JS())
            .pipe(rename(function(path) {
              path.extname = '.min' + path.extname;
            }))
            .pipe(gulp.dest(COMPONENT_DEST));
});

gulp.task('build', ['x-app']);

gulp.task('default',['build'], function() {
  gulp.watch([COMPONENTS_DIR + '/*', COMPONENTS_DIR + '/**/*'], ['build']);
});
```

**NOTE: If you are using gulp-vulcanize you will need to use v5.0.0. Due to a change to support Polymer 0.8 (in alpha at the time of this blog post), gulp-vulcanize v6.0.0 will not correctly inline scripts for Polymer 0.5 components.** 

Some things to note:

- I have opted to have a separate directory for all of my compiled HTML assets -(hence the `COMPONENT_DEST`). Just a personal preferance.
- In each of my components I have utilized relative linking (e.g. `<link rel="import" href="../x-header/x-header.html">`). I have seen this as a pattern in some of the paper components produced by the Polymer team and following the pattern made good sense (This principle extends to both CSS and JS files). It also lends to vulcanization working well and a nice file structure when combined with Bower and I'm all for that!
- After building my compiled component I minify the HTML and the inlined CSS/JS files, change the output file name and pipe the file to its final destination. These are simply steps to reduce the overall file size and identify the component as a minified asset.

## That's all folks!
That's the boilerplate that I work with at the moment. I created a sample repo based on these comments that can be seen on [github](https://github.com/jshcrowthe/polymer-express-boilerplate).
