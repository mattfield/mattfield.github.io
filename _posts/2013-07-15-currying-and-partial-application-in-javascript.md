---
layout: post
title: "Currying and Partial Application in JavaScript"
category: tutorials
tags: [functional programming, javascript, introduction]
---

The concepts of currying and partial application are two programming concepts
that the JavaScript programmer may not be instantly familiar with. This 
would not be surprising given their origins in functional programming and given also
that many articles and posts on JavaScript focus 
primarily on the object-oriented aspects of programming in the language. Arguably, this focus has been to the detriment of 
other programming paradigms and their applicability and usefulness in JavaScript programming. Currying and partial 
application are two such useful concepts.

This post aims to give a fairly high-level overview of currying and partial application in JavaScript. We assume the reader 
has a basic understanding of the nature of functions in JavaScript.

## Partial Application

Partial application can be defined as the process of binding a fixed number of arguments to a function, 
producing another function of smaller arity. This new function can then be called with a reduced number of
arguments than the initial function.

Suppose we have a function `add` that takes two arguments and returns their sum:

{% highlight javascript %}
function add(a, b){
  return a + b;
}
{% endhighlight %}

Which we can of course call by providing both arguments:

{% highlight javascript %}
add(1, 2);
-> 3
{% endhighlight %}

Now, also suppose you have a situation where you need to call our `add` function repeatedly, but provide 
the same first argument _every single time_. You could just go ahead and do this:

{% highlight javascript %}
add(1, 2) // 3
add(1, 10) // 11
add(1, 258) // 259
add(1, 1493022) // 1493023
{% endhighlight %}

However, this is a pretty hefty violation of the [DRY](http://en.wikipedia.org/wiki/Don't_repeat_yourself) principle. 

Instead, what we can do is partially apply a fixed argument to our `add` function, creating a new function with the first argument 
(in this case `1`) bound to it. This new function could then be called repeatedly by supplying only the second argument:

{% highlight javascript %}
// Assuming we have a method called `partial` that performs the application
var addOne = add.partial(1);

addOne(2) // 3
addOne(258) // 259
{% endhighlight %}

Much better! Essentially, the transformation that our `partial` method is performing here returns a new 
function that has every instance of the first argument in the function body replaced with the argument 
we provide to `partial`. In this instance, what `partial` creates is basically:

{% highlight javascript %}
function addOne(b){
  return 1 + b;
}
{% endhighlight %}

The good news is that in ES5 we already have a method available to us that performs partial application 
for us without the need to define our own: `Function.prototype.bind`. [Function#bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind) 
takes a context as it's first argument, followed by an arbitrary list of arguments:

{% highlight javascript %}
myFunc.bind(thisArg, arg1, arg2, ...)
{% endhighlight %}

`Function#bind` does exactly what we want it to; it creates a new function with a given context and a 
sequence of arguments that will precede any other arguments provided when the new function is called. Perfect!

So, we can use it to achieve the same result as above:

{% highlight javascript %}
var addOne = add.bind(this, 1);

addOne(2) // 3
addOne(258) // 259
{% endhighlight %}

We can even use it with functions of higher arity too, since it accepts an arbitary number 
of arguments:

{% highlight javascript %}
function add(a, b, c, d){
  return a + b + c + d;
}

var addOneAndTwoAndThree = add.bind(this, 1, 2, 3);
addOneAndTwoAndThree(4); // 10
{% endhighlight %}

If you enter our partially applied `addOne` function into Chrome Dev Tools, you'll get the following returned:

{% highlight javascript %}
function () { [native code] }
{% endhighlight %}

This is a representation of the new partially applied function that `Function#bind` creates. 

## Currying

Currying is a transformative technique that, when applied to a function with multiple arguments, returns 
a new function composed of nested unary functions that can then be called as a chain:

Let's take our original, uncurried `add` function:

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

This is where a knowledge of functions as first-class citizens in JavaScript comes into place. 

In JavaScript, functions can return other functions. So, our curried function here, when called with one argument, simply returns the contents of its body, namely the anonymous function, and 
stores a reference to the argument in its referencing environment (which the anonymous function can access).
This is why the curried function is invoked as a chain of calls. The first call provides an argument to the first function, 
the second call provides an argument to the second function and so on. Each successive function in the chain has access to all the variables passed to prior function 
calls due to the closure created by the outer-most `add` function. Curried functions will return curried functions if there are still arguments 
remaining that need to be provided for all functions in the chain to be evaluated.

## Linking it all together

What this means is that we can create a curried function, pass in fewer-than-required arguments, store a reference to the 
returned function, and pass it around in our application:

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

................................

## Currying != Partial Application

The terms currying and partial application are sometimes used interchangeably, even though they work differently. It's quite easy to 
see how misconceptions arise given that, at first blush, one could mistake the process in the previous example for partial application. 
The key difference between the two transformations is in what they produce. 

* Currying _always_ produces nested unary functions, with the transformed function still 
being largely the same as the original. 
* The functions produces by partial application, however, are different 
to their original - they need less arguments to invoke.

Following these two points should hopefully help keep the two concepts separate.

## Now What?

Hopefully this whistle-stop tour has imparted some insight into the applicability of currying 
and partial application in your everyday JavaScript programming. There are a whole host of 
libraries out there that will be of use to you when using partial application and currying in 
your JavaScript code, such as:

- [curry](https://github.com/dominictarr/curry)
- Lo-dash's [partial](http://lodash.com/docs#partial), [partialRight](http://lodash.com/docs#partialRight) and [bind](http://lodash.com/docs#bind) methods
- and a whole host of functional programming goodies in the [underscore-contrib repo](https://github.com/documentcloud/underscore-contrib)

................................

## References and Partial Sources

* Ben Alman's [Partial Application in JavaScript](http://benalman.com/news/2012/09/partial-application-in-javascript/)
* Dr. Axel Rauschmayer's [Currying versus partial application](http://www.2ality.com/2011/09/currying-vs-part-eval.html)
* Wikipedia pages on [currying](http://en.wikipedia.org/wiki/Currying) and [partial application](http://en.wikipedia.org/wiki/Partial_application)