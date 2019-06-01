---
title: "Using Polymer WebComponents with Angular.js"
slug: "/using-polymer-webcomponents-with-angular-js"
description: "Polymer and Angular.js are two incredibly popular projects that are run by Google. The two projects serve different purposes yet can, with some coercion, be made to work together."
date: "2015-05-26T02:32:34.000Z"
published: true
categories: 
  - "Polymer"
  - "Web components"
  - "angularjs"
  - "angular"
---

Polymer and Angular.js are two incredibly popular projects that are run by Google. The two projects serve different purposes yet can, with some coercion, be made to work together. Eric Bidelman, a dev from the Polymer team, made a great youtube video on this (which can be found [here](https://www.youtube.com/watch?v=p1NpZ-0Op0w)) and I wanted to give another example of the two projects working side-by-side.

I have posted a [Github repo](https://github.com/jshcrowthe/polymer-angular-demo) that contains all this code and I would encourage you to clone/download the repo and play with it on your own machine.

# `<name-card>` Component
To get started off I am going to declare a component called `<name-card>`. At the time of writing this the latest version of Polymer is `0.9` and so I will be writing modules that way. For simplicity I'm going to use a slightly modified version of the name-card tag that Rob Dodson built in this [polycast](https://www.youtube.com/watch?v=7jolqbtIdiY&list=PLOU2XLYxmsII5c3Mgw6fNYCzaWrsM3sMN).

`name-card.html`
```markup
<link rel="import" href="../polymer/polymer.html">

<dom-module id='name-card'>
  <link rel="import" type='css' href="https://maxcdn.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.min.css">
  <template>
    <p>
      <template is='dom-if' if="{{!edit}}">
        <span>{{fullname}}</span>
      </template>
      <template is="dom-if" if="{{edit}}">
        <label>First:</label> <input type="text" value="{{first::input}}"> <label>Last:</label> <input type="text" value="{{last::input}}">
      </template>
      <button on-click="toggleEdit"><i class="fa fa-pencil"></i></button>
    </p>
  </template>
</dom-module>
<script>
  Polymer({
    is: 'name-card',
    properties: {
      first: String,
      last: String,
      fullname: {
        computed: '_computeFullName(first, last)'
      },
    },
    ready: function() {
      this.edit = false;
    },
    toggleEdit: function() {
      return (this.edit = !this.edit);
    },
    _computeFullName: function (first, last) {
      return first + ' ' + last;
    }
  });
</script>
```

_The only difference between my component and Rob's are the edit button (and associated flag) and the styles. All else is in the polycast above so feel free to check that out if you don't understand a part of this._ 

You'll be able to then instantiate the component very simply as follows!
```markup
<name-card first="Josh" last="Crowther"></name-card>
```

# Sample data binding

Now that we've got an element we can start to play with it! Below is a barebones HTML structure so you can see Angular's data-binding in action!

```markup
<!DOCTYPE html>
<html ng-app>
  <head>
    <title>Demo</title>
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap.min.css" rel="stylesheet">
    <link rel='import' href='/components/name-card/name-card.html' />
  </head>
  <body>
    <div class="container">
      <div class="row">
        <h1>Demo</h1>
        <p>
          <label for="f_name">First Name:</label>
          <input type="text" ng-model="f_name">
        </p>
        <p>
          <label for="l_name">Last Name:</label>
          <input type="text" ng-model="l_name">
        </p>
        <name-card first="{{f_name}}" last="{{l_name}}"></name-card>
      </div>
    </div>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.3.15/angular.min.js"></script>
  </body>
</html>
```

_Make sure that the path for your HTML Import is correct and that, if you are using a browser that doesn't natively support HTML imports, you are including the `webcomponents.js` polyfill._

A working demo of this can be found here: http://jshcrowthe.github.io/polymer-angular-demo/simple.html

This is a simple example of Angular and Polymer working together! Although the example works pretty much as you would expect, some really cool stuff is going on. We are binding two inputs to the `f_name` and `l_name` scope variables and then we are injecting those two attributes into our `name-card` component. Then as those values change Angular is updating the attributes on the name-card component on the fly! In addition to this, the `name-card` component's internal data-binding (given to us by Polymer) is detecting all changes to the first and last name properties and is updating the value of the `fullname` property and the internal template.

Cool huh?

We can even click the little edit affordance in our component and edit the values of `first` and `last`. Polymer will, as we update the values, recalculate the full name property and update the template on the fly! It's awesome.

But for those playing with the code: Did you notice anything odd?

There is something extremely important here to realize: **Any changes made INSIDE a polymer component (NOT Angular.js) do not push changes to Angular.js.**

What this means for us is that when using these two projects out of the box, data-binding is one directional. It flows from Angular to Polymer and NOT the other way around. Any changes made directly inside the Polymer component will not be recognized by Angular AND any updates to the Angular variables being injected into the component will supersede the existing data (even if it was changed by Polymer more recently).

This introduces quite a conundrum doesn't it?

# Getting data binding to flow from Polymer to Angular

We want any changes on the polymer element to also update the Angular scope. After doing some looking I came across an Angular directive called [bindPolymer](https://github.com/eee-c/angular-bind-polymer) that did the job quite nicely. The only downside to this is I have to add a `bind-polymer` attribute to any polymer component that needed to have its internal changes be recognized by Angular.js. In short, it's not a global handler out-of-the-box. However that can also have it's advantages.

In addition to adding this directive we also need to slightly modify our WebComponent to properly update the elements attributes. (I know it's a pain right?)

Here's a demo:

`<name-tag>` component
```markup
<link rel="import" href="../polymer/polymer.html">

<dom-module id='name-card'>
  <link rel="import" type='css' href="https://maxcdn.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.min.css">
  <template>
    <p>
      <template is='dom-if' if="{{!edit}}">
        <span>{{fullname}}</span>
      </template>
      <template is="dom-if" if="{{edit}}">
        <label>First:</label> <input type="text" value="{{first::input}}"> <label>Last:</label> <input type="text" value="{{last::input}}">
      </template>
      <button on-click="toggleEdit"><i class="fa fa-pencil"></i></button>
    </p>
  </template>
</dom-module>
<script>
  Polymer({
    is: 'name-card',
    properties: {
      first: {
        type:String,
        reflectToAttribute: true
      },
      last: {
        type:String,
        reflectToAttribute: true
      },
      fullname: {
        computed: '_computeFullName(first, last)'
      },
    },
    ready: function() {
      this.edit = false;
    },
    toggleEdit: function() {
      return (this.edit = !this.edit);
    },
    _computeFullName: function (first, last) {
      return first + ' ' + last;
    }
  });
</script>
```

And then our Angular App (I simply grabbed the raw bindPolymer directive pardon my shortcut).

```markup
<!DOCTYPE html>
<html ng-app='app'>
  <head>
    <title><%= title %></title>
    <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.4/css/bootstrap.min.css" rel="stylesheet">
    <link rel='import' href='/components/name-card/name-card.html' />
  </head>
  <body>
    <div class="container">
      <div class="row">
        <h1>Demo</h1>
        <p>
          <label for="f_name">First Name:</label>
          <input type="text" ng-model="f_name">
        </p>
        <p>
          <label for="l_name">Last Name:</label>
          <input type="text" ng-model="l_name">
        </p>
        <name-card first="{{f_name}}" last="{{l_name}}" bind-polymer></name-card>
      </div>
    </div>
    <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.3.15/angular.min.js"></script>
    <script>
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

            // When Polymer sees a change to the bound variable,
            // $apply / $digest the changes here in Angular
            var observer = new MutationObserver(function polymerMutationObserver(mutations) {
              scope.$apply(function processMutationsHandler() {
                mutations.forEach(function processMutation(mutation) {

                  var attributeName, newValue, oldValue, getter;
                  attributeName = mutation.attributeName;

                  if(attributeName in attrMap) {
                    newValue = element.attr(attributeName);
                    getter = attrMap[attributeName];
                    oldValue = getter(scope);

                    if(oldValue != newValue && angular.isFunction(getter.assign)) {
                      getter.assign(scope, newValue);
                    }
                  }
                });
              });
            });

            observer.observe(element[0], {attributes: true});
            scope.$on('$destroy', observer.disconnect.bind(observer));
          }
        }
      };
    }]);
    </script>
  </body>
</html>
```

Demo Here: http://jshcrowthe.github.io/polymer-angular-demo/index.html

And now you've got an app that is using a Polymer WebComponent, with the ability to push changes to the hosting angular application.

Ideally I want to revisit this and find/make a way to auto register this binding. That way the data binding will be bi-directional (Angular -> Polymer **AND** Polymer -> Angular) without the need to manually register it on each component. But that will be a later article!

Shoot me a note with any questions/comments as I'd love to hear more of what people are thinking concerning the use of these two projects!


# My opinion (take it or leave it)
I wanted to take just a second to give my personal opinion on doing what I demo'ed here above. 

The tl;dr version of it is:
**Don't do this. Pick one or the other. Not both.**

The extended version:
Polymer and Angular.js are two entirely different approaches to building Web Applications. Although you can get the two to play together (as shown) you will make your life (and more importantly the guy who has to maintain your code's life) much more simple. It's important to note that Angular 2.0 directives and Polymer WebComponents are going to be based on THE SAME THING. Meaning: the underlying standards are the same! You won't have to implement a directive with the sole purpose of getting your components to talk to your application. You will be up and running with even less hastle.

**NOTE: A follow up post to this has been posted [here](http://jcrowther.io/2015/07/13/linking-polymer-model-updates-to-angularjs-digest-cycle/) outlining a lighter solution for the `bindPolymer` directive**



