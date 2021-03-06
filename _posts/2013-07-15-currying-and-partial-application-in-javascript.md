---
layout: post
title: "Currying and Partial Application in JavaScript"
category: javascript
tags: [functional programming, javascript, introduction, tuts]
---

Currying and partial application are two awesome techniques you may not have heard of.  
This post aims to give a fairly high-level overview of both concepts in JavaScript and 
why you should definitely be using them.

## Partial Application

Partial application binds any number of arguments to a function and
produces another function that can be called with fewer arguments than the original.

Suppose we have a function `add` that takes two arguments and returns their sum:

{% highlight javascript %}
function add(a, b){
  return a + b;
}

add(1, 2);
//=> 3
{% endhighlight %}

Let's say you need to call our `add` function repeatedly, but provide 
the same first argument _every single time_. You could just dive in and do this:

{% highlight javascript %}
add(1, 2);
//=> 3
add(1, 10);
//=> 11
add(1, 258);
//=> 259
add(1, 1493022);
//=> 1493023
{% endhighlight %}

But you really shouldn't. Ouch. This has the potential to become unwieldy very quickly.

Instead, we can partially apply one argument to our `add` function, which gives us a new function with that argument
bound to it. We can then call this new function repeatedly by supplying only the second argument. 

Here we're using Lo-Dash's `_.partial` method to perform the partial application:

{% highlight javascript %}
var addOne = _.partial(add, 1);

addOne(2);
//=> 3
addOne(258);
//=> 259
{% endhighlight %}

Much better! What we've created using `_.partial` is essentially:

{% highlight javascript %}
function addOne(b){
  return 1 + b;
}
{% endhighlight %}

We actually already have an ES5 method available to us that performs partial application: `Function.prototype.bind`. The method
takes an arbitrary list of arguments to partially apply to your function. You do additionally have to supply a context (a value for `this`) as the first argument:

{% highlight javascript %}
myFunc.bind(contextArg, arg1, arg2, ...)
{% endhighlight %}

It does exactly what we want it to; creates a new function with a sequence of arguments that will precede any other 
arguments provided when the new function is called. Perfect!

{% highlight javascript %}
var addOne = add.bind(this, 1);

addOne(2)
//=> 3
addOne(258)
//=> 259
{% endhighlight %}

We can use it with functions that take more arguments too, since `Function.prototype.bind` accepts an arbitary number of arguments:

{% highlight javascript %}
function add(a, b, c, d){
  return a + b + c + d;
}

var addOneAndTwoAndThree = add.bind(this, 1, 2, 3);
addOneAndTwoAndThree(4);
//=> 10
{% endhighlight %}

................................

## Currying

A curried function is one that is composed of nested unary functions (functions that take one argument) that can then be called as a chain. 

The process of currying itself isn't all that intuitive, but just to show what the transformation looks like let's take our uncurried `add` function from earlier:

{% highlight javascript %}
function add(a, b){
  return a + b;
};
{% endhighlight %}

In a nutshell, _currying_ is the process of taking a function with multiple arguments and creating another function that:
1) Takes only a single argument, and
1) Returns another curried function if there are any remaining arguments 

So, using these two rules, the curried version of `add` would look like this:

{% highlight javascript %}
function add(a){
  return function(b){
    return a + b;
  }
}
{% endhighlight %}

Let’s break that down:

{% highlight javascript %}
// The outer function `add` takes one argument: `a`
function add(a){
  // which returns an anonymous function that takes
  // a second argument: `b`
  return function(b){
    // This anonymous function then returns the sum 
    // of both arguments
    return a + b;
  }
}
{% endhighlight %}

What this means is that we can invoke our curried function using a chain of function calls...

{% highlight javascript %}
add(1)(2);
//=> 3
{% endhighlight %}

...or, if we supply less than the total number of required arguments, a function is returned that expects some or all of the remaining arguments:

{% highlight javascript %}
// Supplying the first argument to `add`
var addOne = add(1);

// returns an anonymous curried function assigned to `addOne`
//=> addOne = function(b){ return a + b }

// …that’s waiting for a second argument
addOne(2);
//=> 3
{% endhighlight %}

This is a **very** contrived example, but what we’ve just done here should look familiar to you now. It’s partial application, but there’s a slight distinction to be made. We haven’t used any methods to perform the partial application for us here; the structure of the curried function is enough to accomplish it. Currying sets us up perfectly to create functions that can be very easily partially-applied. We’ll go into further detail about how to use currying in just a minute.

Before we do however, let's be honest, no-one would sit and write all this cruft out in any language with higher-order functions and closures. Plus invoking a curried function
with a chain of calls is kinda ugly to look at. To simplify things, we're going to use [curry](https://github.com/dominictarr/curry), 
a simple module that aims to make creating and using curried functions way easier, allowing us to do something like this:

{% highlight javascript %}
var curry = require('curry');
var addThree = curry(function(a, b, c){
  return a + b + c;
});

addThree(1)(2)(3);
//=> 6

// Or, scrapping the ugly
addThree(1, 2, 3);
//=> 6
{% endhighlight %}

Swish.

## Linking it all together

So how is any of this _actually_ useful? The real advantage of these techniques is that they give you the ability to create 
small, reuseable chunks of code that can then be used as building blocks.

Time for a less-contrived example.

Let's make a little wrapper around the modulo operator and a utility function that checks whether a number is odd:

{% highlight javascript %}
var modulo = curry(function(divisor, dividend){
  return dividend % divisor;
});

modulo(3, 9);
//=> 0

// Partially applying `modulo` with the number 2 to give us our utility function
var isOdd = modulo(2);

// Which returns a truthy or falsey value
isOdd(6);
//=> 0
isOdd(5);
//=> 1
{% endhighlight %}

Cool. So, this is pretty useful in it's own right, but let's take things a bit further:

{% highlight javascript %}
// Another wrapper, this time around the native `filter` method
var filter = curry(function(f, xs){
  return xs.filter(f);
});

filter(isOdd, [1,2,3,4,5]);
//=> [1,3,5]

// Partially applying `filter` with `isOdd`
var getTheOdds = filter(isOdd);

// And we end up with a useful little function that will 
// return all the odd numbers in an array
getTheOdds([1,2,3,4,5]);
//=> [1,3,5]
{% endhighlight %}

Now this is why these two techniques are **awesome**. We've created a pretty useful utility function that will 
return all the odd numbers in an array built by using a bunch of small functions as building blocks.

This method of function scaffolding is one of the fundamental techniques in functional programming.

The added benefit of using these techniques is that they encourage you to slow down; think carefully 
about what you're trying to achieve and break the process down into programmatic steps. Then you can create 
discrete blocks of functionality for each of those steps before combining them together. You may even find 
a simpler way of solving old problems in the process.

## References and Partial Sources

* Ben Alman's [Partial Application in JavaScript](http://benalman.com/news/2012/09/partial-application-in-javascript/)
* Dr. Axel Rauschmayer's [Currying versus partial application](http://www.2ality.com/2011/09/currying-vs-part-eval.html)
* Wikipedia pages on [currying](http://en.wikipedia.org/wiki/Currying) and [partial application](http://en.wikipedia.org/wiki/Partial_application)
* ["Hey Underscore, You're Doing It Wrong!"](https://www.youtube.com/watch?v=m3svKOdZijA) - a talk by Brian Lonsdorf at [HTML5DevConf 2013](http://html5devconf.com/)
