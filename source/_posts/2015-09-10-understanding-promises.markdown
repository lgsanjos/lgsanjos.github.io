---
layout: post
title: "Understanding Promises"
date: 2015-09-10 11:19:39 -0300
comments: true
categories: tech 
---

Promise is an specification to handle async calls. There are several ways to solve this problem a very popular solution is using callback arguments. In this article I will introduce promises and do a shallow comparison of both solutions.

<!-- more -->

# The problem

Imagine you need to request some data from a server and once you have the response, you will format this data and display into the screen.

For the sake of simplicity and abstraction, let's consider:
- The method "asyncRequest()" that will request some data from an endpoint.
- Only success case for now, later on we add error handling.

## Using callback

{% codeblock lang:js %}
asyncRequest(function (data) {
  // success 
});
{% endcodeblock %}

## Using promises

{% codeblock lang:js %}
  asyncRequest().then(function (data) {
     // success
  });
{% endcodeblock %}

## Whaat?

At first glance it's not a big change, but looking more carefully in fact what is happening on the promise example is that the asyncRequest is not triggering the callback function instead it is returning an object that implements the method **then**, and this object (which is called promise) will trigger once it's fullfilled.

Let's keep the baby steps and not dive too fast. So now that we have the response we need to validade this data sending this to another service, to do so we will call asynValidation(data).

## Using callback

{% codeblock lang:js %}
asyncRequest(function (data) {
  // asyncRequest succeded 
  asyncValidation(data, function () {
    // asyncValidation succeded
  });
});
{% endcodeblock %}

## Using promises

{% codeblock lang:js %}
asyncRequest().then(asyncValidation)
              .then(function () {
                  // asyncValidation succeded
              });
{% endcodeblock %}

## Whaaaaaaat?

Now we are talking, so the benefit of delegating the event triggering is that you can build a chain of execution, and whatever asycnRequest return will be passed as argument to asyncValidation.

To better undestanding, let's think on a possible implementation of the asyncRequest method.

## Using callback

Ok, this is such a bummer, but I don't want to mess up with any HTTP API, instead I will use a setTimeout to simulate an async process.

{% codeblock lang:js %}
function asyncRequest (successCallback) {
  // do the request using an API
  var response = 'some data';

  setTimeout(function () {
     successCallback(response);
  }, 2000);
}
{% endcodeblock %}

## Using promises

The example below the variable **q** is a promise library, I will get into it below.

{% codeblock lang:js %}
function asyncRequest () {

  // Create a deferred which is the object that will trigger the callback
  var deffered = q.deferred();
  var response = 'some data';

  setTimeout(function () {
    // here we say that the promise should be fullfilled as success
    deffered.resolve(response);
  }, 2000);
 
 return deffered.promise;
}
{% endcodeblock %}

## Enlightening

The variable **q** is the library that implements a promise solution, when called .deferred it will create a _promise manager_, I know that's not the best name, but think that this manager owns the promise and also dictated if it will succeed or fail.

Notice that it returns **deferred.promise**, which is the promise itself. Yet there is no definition of success or fail, it is called **pending promise**.

Once the timeout executes the deferred will trigger **.resolve(response)** which is the way to say, this promise succeded, go on with the chain and use this response variable as parameter to the next method.

It means that as soon as the promise is resolved the **then** method will be invoked and it's function will be called.

## Chaining

Chaining is only possible because the **then** method also retuns a promise, thats will be triggered automatically once it's function is done and it's return value will be carried over to the next function as parameter, so you can do thigs like:

{% codeblock lang:js %}
openSpinnerModal()
         .then(requestData)
         .then(validateData)
         .then(notifyUser)
         .then(closeSpinnerModal);
{% endcodeblock %}

## Failure handling example

Adding a fail scenario to asyncRequest and asyncValidation.

## Using callback

{% codeblock lang:js %}
asyncRequest(
  function (data) {
    // success callback for asyncRequest
   asyncValidation(data, function () {
     // success callback for asyncValidation
     }, function () {
       //fail callback for asyncValidation
     })
  },
  function (exception) {
    // fail callback for asyncRequest
  });
{% endcodeblock %}

Refactoring to a little bit:

{% codeblock lang:js %}
var failCallback = function (exception) {
    // fail callback for asyncRequest
};

var successCallback = function (data) {
   asyncValidation(data, function () {
     // success callback
     }, function () {
       //fail callback
     });
};
{% endcodeblock %}

## Using promises

{% codeblock lang:js %}
asyncRequest().then(asyncValidation, failCallbackForAsyncRequest)
             .then(successForAsyncValidation, failForAsyncValidation);
{% endcodeblock %}

**OR**

{% codeblock lang:js %}
asyncRequest().then(asyncValidation)
              .catch(function (exception) { 
                // Handle both failures here 
              });  
{% endcodeblock %}

## Shortest conclusion ever
 
Chaining is a readable and expressive way to write code, I particularly keep using callbacks for very simple cases but when I notice that it requires me to execute one function after other, I go straight to chaining with promises. 
 
Callbacks inside other callback is a problem well known as **callback hell** this is a good enough reason to give a try for promises. 
 
Also there is much more the **then** and **catch** but it depends on the implementation you will use.
Another useful method is **.finally()** as it sounds like will always execute as the last element of the chain.

----

## Links

Hope you have enjoyed and be curious enough to take a look at the promise definition:
   
- **A+** <http://promisesaplus.com>
 
- And some of it's implementations: 
  - **Kriskowal** q: <https://github.com/kriskowal/q> 
  - **AngularJS** implementation $q: <https://docs.angularjs.org/api/ng/service/$q> 
  - More **references** on: <https://promisesaplus.com/implementations> 

## Extra

The original problem presented here is a piece of a general topic named "Communicating Sequential Process", you can check more if it here <https://en.wikipedia.org/wiki/Communicating_sequential_processes> and also some other solutions other than promises and callbacks are listed below.
