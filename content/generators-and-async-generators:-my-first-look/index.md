---
title: "Generators & Async Generators: My first look"
description: "I recently got on this kick where, I wanted to really try and understand generators (introduced in ES2015), and find some good use cases…"
date: "2019-06-01T21:23:48.364Z"
categories: []
published: false
---

I recently got on this kick where, I wanted to really try and understand generators (introduced in ES2015), and find some good use cases for practical use in client side code. I had some success, and wanted to jot some things down, so here we are.

_NOTE: If you don’t already have a basic idea of what a generator function is, I’d advise taking a look at_ [_MDN_](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators#Generators) _for a quick synopsis._

Per a suggestion from a friend at work, I started looking at how I could use generator functions with pagination. I was surprised with how expressive the code I was able to write was and decided it was worth sharing.

Consider the following code:

It’s a very simple, albeit contrived, example that illustrates how easy this can be. We take some dataset we wish to chunk, we start a tracking variable at 0, and we yield a chunk that is a slice of the array. Take a look at it in action on JSBin.

[**JS Bin**  
_A live pastebin for HTML, CSS & JavaScript and a range of processors, including SCSS, CoffeeScript, Jade and more..._jshcrowthe.jsbin.com](https://jshcrowthe.jsbin.com/xomari/edit?js,console "https://jshcrowthe.jsbin.com/xomari/edit?js,console")[](https://jshcrowthe.jsbin.com/xomari/edit?js,console)

While this is a simple yet powerful concept, it becomes even more useful when paired with async/await. 

“Async Generators” lend themselves very well to the notion of a paginated dataset. For this example I’ll use one of my favorite APIs, PokeApi.co.