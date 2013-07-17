---
layout: post
title: "Currying and Partial Application in JavaScript"
category: javascript
tags: [functional programming, javascript, introduction, tuts]
---

Currying and partial application are two awesome techniques you may not have heard of.  
This post aims to give a fairly high-level overview of both concepts in JavaScript. 

## Partial Application

Partial application binds any number of arguments to a function and
produces another function that can be called with fewer arguments than the original.

Suppose we have a function `add` that takes two arguments and returns their sum:

{% highlight javascript %}
function add(a, b){
  return a + b;
}

add(1, 2);
-> 3
{% endhighlight %}

Let's say you need to call our `add` function repeatedly, but provide 
the same first argument _every single time_. You could just dive in and do this:

{% highlight javascript %}
add(1, 2);
-> 3
add(1, 10);
-> 11
add(1, 258);
-> 259
add(1, 1493022);
-> 1493023
{% endhighlight %}

But you really shouldn't. Ouch. This has the potential to become unwieldy very quickly.

Instead, we can partially apply one argument to our `add` function, which gives us a new function with that argument
bound to it. We can then call this new function repeatedly by supplying only the second argument. 

Here we're using Lo-Dash's `_.partial` method to perform the partial application:

{% highlight javascript %}
var addOne = _.partial(add, 1);

addOne(2);
-> 3
addOne(258);
-> 259
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

addOne(2) // 3
addOne(258) // 259
{% endhighlight %}

We can use it with functions that take more arguments too, since `Function.prototype.bind` accepts an arbitary number of arguments:

{% highlight javascript %}
function add(a, b, c, d){
  return a + b + c + d;
}

var addOneAndTwoAndThree = add.bind(this, 1, 2, 3);
addOneAndTwoAndThree(4); // 10
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

The curried version of `add` would look like this:

{% highlight javascript %}
function add(a){
  return function(b){
    return a + b;
  }
}
{% endhighlight %}

We can then invoke our curried function using a chain of function calls:

{% highlight javascript %}
add(1)(2); // 3
{% endhighlight %}

But, let's be honest, no-one would sit and write all this cruft out in any language with higher-order functions and closures. Plus invoking a curried function
with a chain of calls is kinda ugly to look at. I'd highly recommend checking out [curry](https://github.com/dominictarr/curry), 
a simple module that aims to make creating and using curried functions way easier, allowing us to do something like this:

{% highlight javascript %}
var curry = require('curry');
var addThree = curry(function(a, b, c){
  return a + b + c;
});

addThree(1)(2)(3);
-> 6

// Or, scrapping the ugly
addThree(1, 2, 3);
-> 6
{% endhighlight %}

Swish.

## Linking it all together

So how is any of this _actually_ useful? The real advantage of these techniques is that they give you the ability to create 
small, reuseable chunks of code easily.

Let's make a little wrapper around the modulo operator and a utility function that checks whether a number is odd:

{% highlight javascript %}
var modulo = curry(function(divisor, dividend){
  return dividend % divisor;
});

modulo(3, 9);
-> 0

// Partially applying `modulo` with the number 2 to give us our utility function
var isOdd = modulo(2);

// Which returns a truthy or falsey value
isOdd(6);
-> 0
isOdd(5);
-> 1
{% endhighlight %}

Cool. So, this is pretty useful in it's own right, but let's take things a bit further:

{% highlight javascript %}
// Another wrapper, this time around the native `filter` method
var filter = curry(function(f, xs){
  return xs.filter(f);
});

filter(isOdd, [1,2,3,4,5]);
-> [1,3,5]

// Partially applying `filter` with `isOdd`
var getTheOdds = filter(isOdd);

// And we end up with a useful little function that will 
// return all the odd numbers in an array
getTheOdds([1,2,3,4,5]);
-> [1,3,5]
{% endhighlight %}

Now this is why these two techniques are awesome. We've created a pretty useful utility function that will 
return all the odd numbers in an array, but we've build it using a bunch of small, reusable functions as building blocks.

................................

## Currying != Partial Application

Let's wrap-up by clearing up a common misconception: currying and partial application are not the same thing. The key difference between the two 
transformations is in what they produce:

* Currying _always_ produces nested unary functions, with the transformed function still 
being largely the same as the original. 
* Partial application produces functions that are different 
to their original - they need less arguments to invoke.

................................

## References and Partial Sources

I can't claim all of this post as my own work. Here are some influential external sources:

* Ben Alman's [Partial Application in JavaScript](http://benalman.com/news/2012/09/partial-application-in-javascript/)
* Dr. Axel Rauschmayer's [Currying versus partial application](http://www.2ality.com/2011/09/currying-vs-part-eval.html)
* Wikipedia pages on [currying](http://en.wikipedia.org/wiki/Currying) and [partial application](http://en.wikipedia.org/wiki/Partial_application)
* ["Hey Underscore, You're Doing It Wrong!"](https://www.youtube.com/watch?v=m3svKOdZijA)
