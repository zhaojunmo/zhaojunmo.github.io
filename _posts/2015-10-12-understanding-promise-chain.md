---
layout: post
title: "Understanding Promise Chain"
description: ""
category: 
tags: [JavaScript, promise]
---
{% include JB/setup %}

I updated some of my code immediately after learning Promise from this website: <https://www.promisejs.org/> .

And then I found that my node.js program crashed without throwing errors. It's very difficult to debug if no exception raise. I must used Promise in a wrong way. So I spent some time to learn how Promise Chain works and how to handle errors with Promise.

The followings are things you should know before using Promise.




##1. then() returns new promise

{% highlight javascript %}
function doSomething() {
  return promise()
          .then()
          .then()
          .then();
}
{% endhighlight %}

Everytime you call a then() function, it returns a new promise object. So, the value of function doSomething() is a promise return by the third then(), not by the promise().




##2. Values pass through the chain

{% highlight javascript %}
function promise() {
  return new Promise(function(resolve, reject) {
    setTimeout(function() {
      resolve(1);
    }, 10);
  });
}

function onFulfilled1(result1){
  console.log('result1 is', result1);
  return result1 + 1;
}

function onFulfilled2(result2){
  console.log('result2 is', result2);
  return result2;
}

promise()
  .then(onFulfilled1)
  .then(onFulfilled2);


// Output:
// result1 is 1
// result2 is 2
{% endhighlight %}

promise() passed 1 to onFulfilled1(), and then onFulfilled1() passed 2 to onFulfilled2().




##3. How error pass through the chain
When there is an error that you can't handle in a onRejected() function, you must pass it through the chain so that the error can be handled by other functions.
{% highlight javascript %}
function promise() {
  return new Promise(function(resolve, reject) {
    throw new Error('Bad things')
    setTimeout(function() {
      resolve();
    }, 10);
  });
}

function onFulfilled1(result1){
  console.log('in onFulfilled1()');
}

function onRejected1(err1){
  console.log('in onRejected1()');
  return Promise.reject(err1); // pass err1 to next step
  //equivalent to: throw err1;
  //return err1; //this won't work!! The returning error will be passed to onFulfilled2()
}

function onFulfilled2(result2){
  console.log('in onFulfilled2()');
}

function onRejected2(err2){
  console.log('in onRejected2()', err2.message); //handle err here
}

function onFulfilled3(result3){
  console.log('in onFulfilled3()'); // error had been handled, onFulfilled3 is called
}

promise()
  .then(onFulfilled1, onRejected1)
  .then(onFulfilled2, onRejected2)
  .then(onFulfilled3)


// Output:
// in onRejected1()
// in onRejected2() Bad things
// in onFulfilled3()
{% endhighlight %}





##4. Case that the error vanishes

{% highlight javascript %}
function promise() {
  return new Promise(function(resolve, reject) {
    setTimeout(function() {
      resolve();
    }, 10);
  });
}

function onFulfilled1(result1){
  console.log('in onFulfilled1()');
  throw new Error('Bad things')
}

function onRejected1(err1){
  console.log('in onRejected1()');
}

function onFulfilled2(result2){
  console.log('in onFulfilled2()');
}

promise()
  .then(onFulfilled1, onRejected1)
  .then(onFulfilled2)


// Output:
// in onFulfilled1()
{% endhighlight %}

The error throw in onFulfilled1() won't be handled in onRejected1(). If there is no onRejected function in the following chain, you will never see the error, it just vanishes.

To solve this problem, make sure you call catch() function at the end of the chain to handle the error.

{% highlight javascript %}
promise()
  .then(onFulfilled1, onRejected1)
  .then(onFulfilled2)
  .catch(function(err){
    console.log('handle:', err.message);
  })
{% endhighlight %}

Or, call done() to end this chain. done() is different from then(), done() won't return a promise. done() will throw the error if you didn't handle the error in the promise chain.

{% highlight javascript %}
promise()
  .then(onFulfilled1, onRejected1)
  .then(onFulfilled2)
  .done(function(){
    console.log('done');
  })
{% endhighlight %}



