---
title: "Polymer dom-repeat filtering and sorting"
slug: "/polymer-dom-repeat-filtering-and-sorting"
description: "With all of the great updates that came associated with the Polymer 1.0 release, I wanted to take some time to shed light on one of my favorites."
date: "2015-06-09T14:54:58.000Z"
published: true
categories: 
  - "Polymer"
  - "Web components"
  - "Polymer v1.0"
  - "dom-repeat"
  - "filtering"
  - "sorting"
---

With all of the great updates that came associated with the Polymer 1.0 release, I wanted to take some time to shed light on one of my favorites. 

In Polymer 0.5 we were introduced to the original repeated template:
```markup
<template repeat="{{stuff in items}}">
  ...
</template>
``` 

This allowed us to iterate over a set of items and output the results as DOM for user consumption. With the release of `v1.0` the Polymer team has added some cool new features to the repeated template that make it a lot of fun to use.

# Getting Started with `dom-repeat`

The first thing to note about the new API is that the initialization syntax has changed from Polymer 0.5 (A simple migration guide can be found in [last weeks post](http://jcrowther.io/2015/06/01/upgrading-from-polymer-v0-5-v1-0/))! So to create the same repeated template as above the syntax would change to be:
```markup
<template is="dom-repeat" items={{items}} as="stuff">
  ...
</template>
``` 

There are a couple changes that were made.
1. Added the attribute `is="dom-repeat"`. This is required! This tag will literally do nothing without this attribute so don't forget it.
2. Changed `repeat` attribute to `items`. Right now you do not specify the alias for the items in the `items` declaration. You simply put the array of values that you want repeated in as the `items` attribute.
3. Utilized `as='item'` attribute to specify alias. By default, if no alias is assigned, the individual item will be bound to the `item` variable inside of a `dom-repeat`. You can change this by changing the `as` attribute and then you can data-bind onto whatever you'd like.

# Filter/Sort Attributes
Now that we've got at list of items that we can work with it's time to shine a light on these two attributes. They are documented in full [in the Polymer documentation](http://polymer.github.io/polymer/) so feel free to hop over there and read up on their use (you'll have to change the select box in the top left to `dom-repeat`).

These two attributes mimic the native Array filter/sort functions so make sure you are familiar with those before getting started!

I have created demo's of the following examples they can be found [on Github](http://jshcrowthe.github.io/dom-repeat-demo/).

## Filtering
This attribute accepts a function directly or a string representing a function on the element itself. To use the filter attribute we need a function that will do our filtering and after we have that simply passing that function to the `dom-repeat` tag will do the rest. Let's take a look.

**Filter Function**
```javascript
...
_isDoctor: function(person) {
  if (!person) return false;
  return person.name && ~person.name.indexOf(('Dr.'));
};
...
```

**Markup**
```markup
<template is="dom-repeat" items="{{list}}" as="card" filter="_isDoctor">
  ...
</template>
```

_[Demo](http://jshcrowthe.github.io/dom-repeat-demo/demo.html)_

This will initially filter the list so only people with "Dr." in the name property will be displayed! 

_NOTE: We can then use the `observe` attribute on the `dom-repeat` and specify properties to watch on the child elements and, if they change, refilter/resort the list. I haven't used this yet_

For my use cases though I needed something that could filter the list as other values on the page change (think categories, search fields, etc). So to achieve that instead of just passing a string that represents a function I actually pass a function directly. I can then customize that function (using other data on the page) and the list will be refiltered as those values change!

The following example would allow us to filter a list of people as a search value on the page changes:

**Filter Function**
```javascript
...
_filter: function(val) {
  return function(person) {
    if (!val) return true;
    if (!person) return false;
    return (person.name && ~person.name.indexOf(val));
  };
}
...
```

**Markup**
```markup
...
<input type="text" value="{{filterVal::input}}">
...
<template is="dom-repeat" items="{{list}}" as="card" filter="{{_filter(filterVal)}}">
  ...
</template>
```

_[Demo](http://jshcrowthe.github.io/dom-repeat-demo/demo2.html)_

And this works nicely! As people change the value of the text box the list will filter to only contain those people with names that contain the value of the textbox! It's pretty slick.

## Sorting
Sorting pretty much follows the same pattern as filtering does except it uses [Array.sort()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort). I'll get right to the examples! 

Say for example we wanted to sort a list of people by their email provider (gmail/yahoo/hotmail) and then alphabetically within the provider. We could execute that doing the following:

**Sort Function**
```javascript
...
_sort: function(a, b) {
  var regex = /@(.*)/;
  var a_provider = a.email.match(regex)[1];
  var b_provider = b.email.match(regex)[1];
  if (a_provider == b_provider) {
    if (a.email === b.email) return 0;
    return a.email < b.email ? -1 : 1;
  }
  return a_provider < b_provider ? -1 : 1;
}
...
``` 

**Markup**
```markup
<template is="dom-repeat" items="{{list}}" as="card" sort="_sort">
  ...
</template>
```

_[Demo](http://jshcrowthe.github.io/dom-repeat-demo/demo.html)_

And with that all of a sudden out list of people is sorted by email provider alphabetically! You can also build an implementation that will allow you to toggle that sort (similar to what we did up above with the searchable filter) by passing a function that returns a function!

So back to our people list example say we wanted to have a couple filters (name, email, email provider) and not just the one. We could accomplish that by setting up a sort function that returns a function and then passing in the current sort value!

**Sort Function**
```javascript
...
_sort: function(val) {
  switch(val) {
    case 'name':
      return function(a, b) {
        if (a.name === b.name) return 0;
        return a.name < b.name ? -1 : 1;
      };
    case 'email':
      return function(a, b) {
        if (a.email === b.email) return 0;
        return a.email < b.email ? -1 : 1;
      };
    case 'email_provider':
      var regex = /@(.*)/;
      return function(a, b) {
        var a_provider = a.email.match(regex)[1];
        var b_provider = b.email.match(regex)[1];
        if (a_provider == b_provider) {
          if (a.email === b.email) return 0;
          return a.email < b.email ? -1 : 1;
        }
        return a_provider < b_provider ? -1 : 1;
      };
  }
}
...
```

**Markup**
```markup
...
<select value="{{sortVal::change}}">
  <option value="name">Name</option>
  <option value="email">Email</option>
  <option value="email_provider">Email Provider</option>
</select>
...
<template is="dom-repeat" items="{{list}}" as="card" sort="{{_sort(sortVal)}}">
  ...
</template
```

_[Demo](http://jshcrowthe.github.io/dom-repeat-demo/demo2.html)_

And just like that, as we toggle the dropdown, we change the sorting of the list. 

These are some incredibly useful features that `dom-repeat` is now capable of! The filtering/sorting are super responsive (they will re-use existing DOM nodes as it filters/sorts the list to minimize additions/removals from the DOM) and take little time to get set up. Thanks to the Polymer team for two more great features!


As always feel free to ping me with questions/comments!
