---
title: "Upgrading from Polymer v0.5 to v1.0"
description: "Last week I was able to attend a Google I/O extended in Bluffdale Utah and it was awesome. There were a ton of cool things announced in the Android space but for me the best part was, by far, the announcing of Polymer 1.0!"
date: "2015-06-01T07:02:20.000Z"
categories: 
  - "Polymer"
  - "Polymer v0.5"
  - "Polymer v1.0"
  - "Polymer 0.5"
  - "Polymer 1.0"
  - "Migration"
published: true
---

Last week I was able to attend a Google I/O extended in Bluffdale Utah and it was awesome. There were a ton of cool things announced in the Android space but for me the best part was, by far, the announcing of Polymer 1.0!

The Polymer team has done a lot of great work improving on Polymer v0.5 and as with any good major version update there are some breaking changes that developers need to be aware of. 

I recently went through the process of upgrading one of my applications (yes, I know the announcement was two days ago... I was excited) and wanted to share some of my learnings as well as the steps I took.

_I put together a simple example illustrating some of the points that I will touch on [here](). Feel free to take a look at the code and shoot me questions if there are any. (I used the only interesting, no set-up required API I could find. I was originally planning on using the Github API and creating a nice Github card but because you can no longer call their API without an OAUTH token I opted to just go with something simple)_

# Migration Steps

Just so everyone is clear, **[read the docs/migration guide](https://www.polymer-project.org/1.0/docs/migration.html)**! I look at them daily and they are really well done. That said there is a lot there so this is meant to be an overview of a few of the changes. So let's get started!

## Goodbye `<polymer-element>`... Hello `<dom-module>`
First and foremost the means by which we declare elements has changed! The `<polymer-element>` tag that was introduced in `v0.5` is gone and has been replaced by the `<dom-module>` tag. In fact, in regards to element registration, there has been a structural shift.

Before we would have done something like the following:

```markup
<polymer-element name="ELEMENT-NAME" attributes="">
  <template>
    <style>
      :host {
        display:block;
      }
    </style>
  
  </template>
  <script>
    Polymer();
  </script>
</polymer-element>
```

Whereas now we declare elements like this:

```markup
<dom-module id='ELEMENT-NAME'>
  <style>
    :host {
      display:block;
    }
  </style>
  <template>
    
  </template>
</dom-module>
<script>
  Polymer({
    is: 'ELEMENT-NAME',
    properties: {
      
    }
  });
</script>
```

A couple key differences.

1. `polymer-element` changed to `dom-module`
2. The name of the element (in this case `ELEMENT-NAME`) was originally registered by the `name` attribute on the `polymer-element` tag. Now that has been changed to the `id` attribute **IN ADDITION** to the `is` property passed into the `Polymer()` function call. These two tags **MUST BE THE SAME**. Otherwise things will break.
3. Since we are talking about the `Polymer()` function call, notice that it has been moved outside of the element we are creating.
4. Declaring attributes has also changed in `v1.0` before we could use the `attributes` attribute or the `publish` object to register attributes. In `1.0` we now use the `properties` property to declare our attribute types in addition to defaults, read-only settings and more. (For a comprehensive list here is a [link to the docs](https://www.polymer-project.org/1.0/docs/devguide/properties.html). Enjoy!)
5. The `<style>` tag now lives inside of the `dom-module` but **NOT** inside of the template as we did before! If you have been using an external stylesheet in your components you will also need to make the move to using HTML imports for your CSS (doc explanation [here](https://www.polymer-project.org/1.0/docs/devguide/styling.html#external-stylesheets)). Simply put, this: `<link rel='stylesheet' href='styles.css'>` would become `<link rel='import' type='css' href='styles.css'>` and you're good to go.


## Data Binding
This has changed a bit. The principle is the same but we are currently limited in the things we can do. The [data-binding section](https://www.polymer-project.org/1.0/docs/migration.html#data-binding) of the official Polymer migration guide explains it the best. But it bears repeating:

**No more expressions or filters**: This makes me sad and I hope we will see it again. So what that means is, only properties or functions defined in the object you pass to `Polymer()` can be data bound to.

**Any binding must make up the entire body of a tag/attribute**: So this means no more: `<p>Hello {{name}}</p>`. The way around this is to either use a `computed` property (read about them on the docs link above) or be clever about your tags (for example utilizing `<span>` tags like so: `<p>Hello <span>{{name}}</span></p>`)

**One way binding:** In addition to the `{{}}` syntax we now also have the `[[]]` syntax! The curly-braces will do 2-way data-binding while the square-brackets will do 1-way! It's a nice to have but cool all the same.

**No more whitespace in binding:** This is in bold in the docs but I TOTALLY missed it. I typically like to do bindings with an extra space like so: `{{ var }}` but this is invalid in `v1.0`. Lame.

## `<template>` changes
Polymer v0.5 gave us 3 very cool uses for the `template` tag. 

1. `<template if="{{condition}}>...</template>`
2. `<template repeat="{{item in items}}">...</template>`
3. `<template is="auto-binding">...</template>`

These have changed ever so slightly. The new way to instantiate them is, respectively:

1. `<template is="dom-if" if="{{condition}}">...</template>`
2. `<template is="dom-repeat" items="{{items}}" as="item">...</template>`
3. `<template is="dom-bind">...</template>`

The API around the `dom-repeat` template has changed but the [template reference](https://www.polymer-project.org/1.0/docs/devguide/templates.html) explains the new API well. Pay special attention to the Array mutators that are required to modify a `dom-repeat`'ed array. I missed those.

## Styling
In Polymer 0.5 element templates were inserted into the shadow DOM of a page. This allowed us to style individual tags (`<h1>`, `<p>`, etc) or classes specific to our component. Due to the nature of shadow DOM the component styles would not bleed into (and conflict with) the existing elements and styles on the page. With the release of Polymer 1.0 the Polymer team has introduced "shady DOM", a much lighter implementation of local DOM that does not require any additional polyfills (yay performance!). Shadow DOM can still be used should you desire ([see the docs for how to turn it on](https://www.polymer-project.org/1.0/docs/devguide/settings.html)), but right now the default across all browsers is shady DOM. With shady DOM ALL of your element templates are simply DOM. Let me say that again: Your elements templates are normal DOM. This means that global stylesheets now apply to your elements so keep that in mind as you are doing your styling. Your elements styles will take priority over existing stylesheets as usual but just keep in mind that you may have to override some global styles in your elements. However, thanks to style scoping you can still style your components at a tag level and the styles will not bleed into other elements.

Also related to styling is that layout attributes have been moved! So to illustrate:

```markup
<div layout horizontal justified>
  ...
</div>
```

No longer does what you'd expect. The attributes have been converted to a stylesheet that you must include in your element (via HTML imports) if you want it. The layout styles are now located in [their own repo](https://github.com/Polymer/layout) on Github and therefore you must `bower install` them separately. After you install and import `layout/layout.html` into your element you can then use the attributes as classes in your elements like so:

```markup
<div class="layout horizontal justified">
  ...
</div>
```

# Fin
These are just a couple of the many changes made but they represent the vast majority of the pain points I experienced as I moved from `v0.5` to `v1.0`. Of course there are other changes (like the change from "core" elements to "iron" elements) but the API's have remained largely the same albeit there are a few modifications. Just make sure that you double check the docs on the element you want to use before you go string replacing `core-` with `iron-`.

I mentioned above that I put together a small repo demonstrating a migration of an element (it's even pokemon themed) so feel free to [check it out on Github](http://jshcrowthe.github.io/polymer-migration/)! It illustrates all of the above as well as some changes in the `iron-` elements so feel free to look around.

Shoot me a question/comment if there are any and I wish you a Happy Migration!
