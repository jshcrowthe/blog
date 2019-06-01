---
title: "Linking Polymer Model Updates to AngularJS Digest Cycle"
slug: "/linking-polymer-model-updates-to-angularjs-digest-cycle"
description: "A few weeks ago I wrote a post on [Using Polymer WebComponents with Angular.js. I had the opportunity to revisit the post, wanted to share my thoughts"
date: "2015-07-13T02:24:39.000Z"
published: true
categories: 
  - "Polymer"
  - "angularjs"
  - "angular"
  - "Polymer v1.0"
  - "Polymer 1.0"
  - "digest"
  - "data-binding"
---

A few weeks ago I wrote a post on [Using Polymer WebComponents with Angular.js](http://jcrowther.io/2015/05/26/using-polymer-webcomponents-with-angular-js/). I had the opportunity to revisit the post and create a component that uses Polymer's [property change notification system](https://www.polymer-project.org/1.0/docs/devguide/data-binding.html#property-notification). This allows us to create a more performant version of the same codebase! 

# `bindPolymer` Directive

In the [previous post](http://jcrowther.io/2015/05/26/using-polymer-webcomponents-with-angular-js/) that I wrote regarding using these two frameworks together we utilized the [bindPolymer](https://github.com/eee-c/angular-bind-polymer) element.

The `bindPolymer` directive has some pretty cool things going on inside of it:

- Parsing attributes looking for data-binding
- Register a `MutationObserver` for each property that is data-bound (Looking for the `{{}}` syntax)
- On a change inside of the `MutationObserver` push those values to the angular scope and trigger a `scope.$apply()`
- Set up listeners to clear out unneeded `MutationObservers` on element removal

Per a suggestion in the comments of my last post I decided to make some changes.

# The new approach
The main thing that I removed when revisiting this was the use of the `MutationObserver`. Per a suggestion in the comments on my last post, I opted to use the `<property>-changed` notifications that are built in to Polymer attributes. My revised solution utilizes the attribute parsing from the original `bindPolymer` element but then utilizes the built in events system to handle `scope` updates as well as the listener teardown.

# The Code
What everyone has been waiting for right?

## Attribute Parsing
This portion is reused from the original `bindPolymer` element. Because we are utilizing the `compile` directive property we are able to access the raw directive template and tell which values we need to set up bindings for.
```javascript
...
compile: function bindPolymerCompile(el, attr) {
  var attrMap = {};
  for (var prop in attr) {
    if (angular.isString(attr[prop])) {
      var _match = attr[prop].match(/\{\{\s*([\.\w]+)\s*\}\}/);
      if (_match) {
        attrMap[prop] = $parse(_match[1]);
      }
    }
  }
  ...
}
...
```
We parse through all of the attributes looking for those using the binding (`{{}}`) syntax. For each of those we map the angular `scope` value to the property value for later usage.

## Setting up the Event Listeners
This is where things change. After populating the `attrMap` (shown above) we then loop through each of the keys and create an event listener looking for `key + '-changed'` (see the [Polymer docs](https://www.polymer-project.org/1.0/docs/devguide/data-binding.html#property-notification) for more on property change notifications).

Each event then enters a `scope.$evalAsync` block that will trigger an angular `$digest` after the contents have all been executed (we use `scope.$evalAsync` to prevent potential errors from being in the middle of a `$digest`). The scope value is then assigned using the `scope` mappings we grabbed in the directive `compile` step. 
```javascript
compile: function bindPolymerCompile(el, attr) {
  var attrMap = {};
  ...
  return function bindPolymerLink(scope, element, attrs) {
    Object.keys(attrMap).forEach(function(key) {
      element.on(key + '-changed', function(event) {
        scope.$evalAsync(function() {
          if (attrMap[key](scope) === event.detail.value) return;
          attrMap[key].assign(scope, event.detail.value);
        });
      });
    });
  };
}
```

## All together
The final product looks like so:

```javascript
angular
.module('app', [])
.directive('bindPolymer', ['$parse', function($parse) {
  return {
    restrict: 'A',
    scope : false,
    compile: function bindPolymerCompile(el, attr) {
      var attrMap = {};
      for (var prop in attr) {
        if (angular.isString(attr[prop])) {
          var _match = attr[prop].match(/\{\{\s*([\.\w]+)\s*\}\}/);
          if (_match) {
            attrMap[prop] = $parse(_match[1]);
          }
        }
      }
      return function bindPolymerLink(scope, element, attrs) {
        Object.keys(attrMap).forEach(function(key) {
          element.on(key + '-changed', function(event) {
            scope.$evalAsync(function() {
              if (attrMap[key](scope) === event.detail.value) return;
              attrMap[key].assign(scope, event.detail.value);
            });
          });
        });
      };
    }
  };
}]);
```

A working demo of this can be found here: http://jshcrowthe.github.io/polymer-angular-demo/polymer-events.html _(NOTE: This is a copy of the example element used in my [previous post](http://jcrowther.io/2015/05/26/using-polymer-webcomponents-with-angular-js/) if you don't understand what's going on head on back and read that post for some context)_

Good luck and shoot me any comments/questions you have!
