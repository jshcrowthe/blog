---
title: "Angular.js (1.x) $http interceptors"
slug: "/angular-js-http-interceptors"
description: "I have been using Angular.js for a while now and of all of the features that exist, the one that I have the most fun using is the $http interceptor!"
date: "2015-05-19T06:08:20.000Z"
published: true
categories: 
  - "angularjs"
  - "angular"
  - "$http"
  - "HTTPInterceptors"
  - "Interceptors"
  - "XMLHttpRequest"
---

I have been using Angular.js for a while now and of all of the features that exist, the one that I have the most fun using is the $http interceptor!

# $http interceptors?

As we all know Angular's `$http` service is simply a wrapper around the `XMLHttpRequest` available in the browser. We utilize the `$http` service like so:

```javascript
$http({
  method: 'GET',
  url: '/api/url'
})
.success(function(data) {
  // Handle Response
})
.error(function(err) {
  // Handle Errors
});
```

This `$http()` method accepts an object containing a bunch of properties that allow us to further configure the request (see a comprehensive list on the [Angular Docs](https://docs.angularjs.org/api/ng/service/$http)). 

This works REALLY well for the vast majority of use cases. However what if I want to do some global configuration? I can pass those configurations into each individual request OR I can use the $http Interceptor!

# Interceptor 101
An extremely common use case for $http interceptors is standardizing headers (think `Authorization` headers). There are only two steps to creating an $http interceptor:
1. Create a factory that returns an object with config functions for the different interceptable events (`request`, `requestError`, `response`, `responseError`)
2. Push this factory onto the `$httpProvider.interceptors` array.


Let's give you a simple example of this in action:

```javascript
angular
.module('demoApp', [])
.factory('authInterceptor', function() {
  return {
    request : function(config) {
      config.headers.authorization = 'Bearer AUTH-TOKEN';
      return config;
    }
  };
})
.config(function($httpProvider) {
  $httpProvider.interceptors.push('authInterceptor');
})
```

Now within the `demoApp` application any request that is made will automatically have the authorization header configured!

Pretty slick right? Let's take it a step further and do some filtering inside of our interceptor. 

Hopefully nowadays everyone is using `json` as their primary means of data transport however every now and again you will come across some archaic API that is still using XML! Let's set up an interceptor that will handle these endpoints.

```javascript
angular
.module('demoApp', [])
.factory('responseInterceptor', function() {
  return {
    response : function(resObj) {
      if (~resObj.headers('content-type').indexOf('application/xml')) resObj.data = xml2json(resObj.data);
      return resObj;
    }
  };
})
.config(function($httpProvider) {
  $httpProvider.interceptors.push('responseInterceptor');
})
```

Now all requests that come into our `demoApp` that come with a content type of `application/xml` will be run through `xml2json()` and have the body redefined so we can use it easily in our services.

# Retry service using interceptors

This is my personal favorite use of interceptors. Despite all services desire to have 100% uptime the occasional 503 is unavoidable! So rather than immediately failing and showing an error state why not retry the request? But to be able to do this on a global scale and define a retry strategy that your entire app (or whatever requests you limit it to) subscribes to is awesome! The best part: because you handle the retry logic in an interceptor your calls to `$http` **DO NOT CHANGE**. You literally have to change zero code and your application will return to the same point in your code whether it takes 1 or multiple requests.

```javascript
angular
.module('app', [])
.factory('retryInterceptor', function($q, $injector) {
  return {
    responseError: function(rejection) {
      if (rejection.status !== 503) return $q.reject(rejection);
      if (rejection.config.retry) {
        rejection.config.retry++;
      } else {
        rejection.config.retry = 1;
      }

      if (rejection.config.retry < 5) {
        return $injector.get('$http')(rejection.config);
      } else {
        return $q.reject(rejection);
      }
    }
  };
})
.config(function($httpProvider) {
  $httpProvider.interceptors.push('retryInterceptor');
})
```

I check if the request failed with a `503` status and if not reject normally. If it is a failure I simply check the `config` object for a `retry` property and then either return a new `$http` request or reject the request (which causes the existing `error` function to fire.

_NOTE: I have to utilize `$injector.get('http')` as Angular will throw an error if I try to include $http the standard way it results in a circular dependency error_

# Wrapping up 
As I said before: **I LOVE INTERCEPTORS!** You can accomplish some pretty awesome stuff with them and make even more powerful angular applications.

I have put together a [small github repo](https://github.com/jshcrowthe/HTTPInterceptor-demo) with these examples that you can feel free to clone and play with. 

As always I look forward to questions/comments! 
