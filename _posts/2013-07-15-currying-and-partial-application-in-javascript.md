---
layout: post
title: "Currying and Partial Application in JavaScript"
category: javascript
tags: [functional programming, javascript, introduction, tuts]
---

Currying and partial application are two awesome techniques you may not have heard of.  
This post aims to give a fairly high-level overview of both concepts in JavaScript. 

## Partial Application

Partial application is the process of binding a fixed number of arguments to a function, 
producing another function of smaller arity that can be called with a reduced number of
arguments.

To illustrate, suppose we have a function `add` that takes two arguments and returns their sum:

{% highlight javascript %}
function add(a, b){
  return a + b;
}

add(1, 2);
-> 3
{% endhighlight %}

Now, also suppose you have a situation where you need to call our `add` function repeatedly, but provide 
the same first argument _every single time_. You could just dive in and do this:

{% highlight javascript %}
add(1, 2) // 3
add(1, 10) // 11
add(1, 258) // 259
add(1, 1493022) // 1493023
{% endhighlight %}

But you really shouldn't. Ouch. This has the potential to become unwieldy very quickly.

Instead, what we can do is partially apply a fixed argument to our `add` function. This gives us a new function with the first argument 
(in this case `1`) bound to it, which can then be called repeatedly by supplying only the second argument:

{% highlight javascript %}
// Assuming we have a method called `partial` that performs the application
var addOne = add.partial(1);

addOne(2) // 3
addOne(258) // 259
{% endhighlight %}

Much better! In this instance, what `partial` creates is essentially:

{% highlight javascript %}
function addOne(b){
  return 1 + b;
}
{% endhighlight %}

The good news is that in ES5 we already have a method available to us that performs partial application `Function.prototype.bind`. 
[Function#bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) 
takes a context as it's first argument, followed by an arbitrary list of arguments:

{% highlight javascript %}
myFunc.bind(thisArg, arg1, arg2, ...)
{% endhighlight %}

It does exactly what we want it to: creates a new function with a sequence of arguments that will precede any other 
arguments provided when the new function is called. Perfect!

{% highlight javascript %}
var addOne = add.bind(this, 1);

addOne(2) // 3
addOne(258) // 259
{% endhighlight %}

We can even use it with functions of higher arity too, since it accepts an arbitary number of arguments:

{% highlight javascript %}
function add(a, b, c, d){
  return a + b + c + d;
}

var addOneAndTwoAndThree = add.bind(this, 1, 2, 3);
addOneAndTwoAndThree(4); // 10
{% endhighlight %}

................................

## Currying

Currying is the process of transforming a function with multiple arguments into 
a new function composed of nested unary functions that can then be called as a chain:

Using our uncurried `add` function from earlier:

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

We can then call our function using a chain of function calls, rather than supplying all required arguments in 
one function call:

{% highlight javascript %}
add(1)(2); // 3
{% endhighlight %}

Let's break down our curried `add` function to explain what's going on:

{% highlight javascript %}
// Create a function `add` that takes one argument
function add(a){
  // Return another function that takes one argument
  return function(b){
    // Return the sum of the both of the functions arguments
    return a + b;
  }
}
{% endhighlight %}

What happens when we invoke `add` with less than two arguments? The result actually helps to explain 
how this curried function behaves:

{% highlight javascript %}
add(1);
-> function(b){
     return a + b;
   }
{% endhighlight %}

A curried function, when called with less than the total number of required arguments, will simply return the next curried function.

## Linking it all together

What this means is that we can create a curried function, pass in fewer-than-required arguments, store a reference to the 
returned function, and pass it around:

{% highlight javascript %}
// Our `add` function from earlier
function add(a){
  return function(b){
    return a + b;
  }
}

var addOne = add(1);

// Remember, calling a curried function with less-than-required arguments 
// return the next function in the chain
addOne();
-> function(b){
     return a + b;
   }

// We can now call `addOne` with an argument to give us the same expected result
addOne(2)
-> 3
{% endhighlight %}

It's been noted that creating curried functions in this verbose manner is a bit of a pain, is tedious and hella ugly. [Curry](https://github.com/dominictarr/curry) is 
a simple module that aims to make the process much easier. Check out [this post](http://hughfdjackson.com/javascript/2013/07/06/why-curry-helps/) by Hugh FD Jackson for the lowdown on 
how and why you'd want to use it.

................................

## Currying != Partial Application

Let's wrap-up by clearing up a common misconception: currying and partial application are not the same thing. The key difference between the two 
transformations is in what they produce:

* Currying _always_ produces nested unary functions, with the transformed function still 
being largely the same as the original. 
* Partial application produces functions that are different 
to their original - they need less arguments to invoke.

................................

## Now What?

Hopefully this whistle-stop tour has provided some insight into the applicability of currying 
and partial application. The real advantage of using currying and partial application is that 
they give you the ability to create small, reuseable chunks of code without excess cruft.

................................

## References and Partial Sources

* Ben Alman's [Partial Application in JavaScript](http://benalman.com/news/2012/09/partial-application-in-javascript/)
* Dr. Axel Rauschmayer's [Currying versus partial application](http://www.2ality.com/2011/09/currying-vs-part-eval.html)
* Wikipedia pages on [currying](http://en.wikipedia.org/wiki/Currying) and [partial application](http://en.wikipedia.org/wiki/Partial_application)
