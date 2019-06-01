---
title: "Internationalization (i18n) and Polymer Web Components"
description: "As I have been working with Web Components solving the implementation of internationalization has been a unique challenge! I figured it merited some further discussion."
slug: "/i18n-and-web-components"
date: "2015-05-11T01:25:44.000Z"
published: true
categories: 
  - "Polymer"
  - "Web components"
  - "i18n"
  - "internationalization"
  - "gulp"
  - "grunt"
  - "translation"
  - "mixins"
  - "mixin"
---

As I have been working with Web Components solving the implementation of internationalization has been a unique challenge! I figured it merited some further discussion.

# The Problem (Component Specific `i18n`)
There are many javascript i18n frameworks available that solve i18n at a page level ([Polyglot](http://airbnb.github.io/polyglot.js/), [i18next](http://i18next.com/), [Jed](http://slexaxton.github.io/Jed/)). However in _my_ ideal world I would love to have a component that could be distributed with all the associated translations and the means for serving them all wrapped into one convenient little bundle. No need to include other libraries or anything.

Unfortunately I haven't found anything like this _(yet...)_  so I decided I'd try implementing something myself.

# My Solution 
As I pondered how to solve this I couldn't think of an easy way to get the raw translations (in my case stored as JSON files) into each web component. Because of this I decided to introduce a build step before the javascript implementation in order to keep things modular.

## The Build Step
Conceptually this step is quite simple: I need to inject the JSON translation files into the web component in a way that our component can access them. The following are two approaches to injecting the translations.

### `Locales.js` file inclusion
**File Structure**
```
- WEB_COMPONENT
  - WEB_COMPONENT.html
  - locales.js
  - locales
    - WEB_COMPONENT_en.json
    - WEB_COMPONENT_es.json
    - WEB_COMPONENT_ko.json
    - WEB_COMPONENT_ja.json
    - WEB_COMPONENT_zh.json
```

**web_component.html**
```markup
<polymer-element name="translated-component">
  <!-- locale inclusion script -->
  <script src="locales.js"></script>
  <template>
    <style> /*...*/ </style>
  </template>
  <script>
    Polymer();
  </script>
</polymer-element>
```

**locales.js**
```javascript
// Before these lines define and include some sort of i18n framework 
// OR an ultra simple object that handles translation. 
var translations = {
  en: require('web_component_en.json'),
  es: require('web_component_es.json'),
  ko: require('web_component_ko.json'),
  ja: require('web_component_ja.json'),
  zh: require('web_component_zh.json'),  
};

// Then inject the translations var into the predefined translation
// service under the namespace of the CURRENT COMPONENT.     
// var translations = {  
//   COMPONENT_NAME: translations, 
//   ...        
// };    
```

The `locales.js` file could then be browserified (or those `require` statements handled some other way) and the translations object should be good to go.

### Direct Inject Method
This method is extremely similar to the previous file inclusion. Instead of bringing in the translations using a `locales.js` file the translations would be injected at build time using a [Gulp](http://gulpjs.com/)/[Grunt](http://gruntjs.com/) task. Picture the above diagrams only minus the locales.js file inclusion.

## Component `i18n` function
After including the translations on the page a simple function would be implemented that would handle the namespaced component access (This could be standardized).

```javascript
var i18nMixin = {
  i18n: function(key) {
    // Access the injected translations here based on this.nodeName
  }
};
```

The original element could be extended as follows to include this functionality:

```markup
<polymer-element name="translated-component">
  <!-- locale inclusion script -->
  <script src="locales.js"></script>
  <template>
    <style> /*...*/ </style>
  </template>
  <script>
    Polymer(Polymer.mixin({
      /* Original Element Config */
    }, i18nMixin));
  </script>
</polymer-element>
```

Within the components you can now access that specific components translations with the `i18n` function.

```markup
<p>{{ i18n('translation_key'); }}</p>
```
OR with the `i18n` filter!
```markup
<p>{{ 'translation_key' | i18n }}</p>
```

The component could then contain, in the .bowerrc file, a postinstall script (assuming you installed via `bower`) that will automatically handle the i18n build process so that the component will be ready to go at runtime!

**The end**

_Please feel free to shoot me a comment with any suggestions/questions I'd love to see what other solutions people have come up with!_
