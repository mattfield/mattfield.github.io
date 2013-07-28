---
layout: post
title: "Taking Things Out of Context: Functors in JavaScript"
category: javascript
tags: [functional programming, javascript, introduction, tuts]
---

## Values and Contexts

We know we can take a value:

{% highlight javascript %}
5
{% endhighlight %}

And we can apply a function to it:
    
{% highlight javascript %}
function addOne(a) { return a + 1; };
addOne(5);
//=> 6
{% endhighlight %}

Pretty straightforward, right? What we also need to realise here though is that our value exists inside a "context". An 
easy way to think of a context is it being like a box that our value sits inside. So what happens if we 
change the context of our value by, say, wrapping it in an array, and then we try to call our 
`addOne` function on it again?

{% highlight javascript %}
addOne([5]);
//=> 51
{% endhighlight %}

Say what?

Turns out we're going to end up with different results when we apply our function to a value depending 
on the value's context. Our `addOne` function works just fine when we provide a value in the context it expects, 
but when we change that context everything goes wrong. We're still dealing with the same value with the same data 
type though, it hasn't fundamentally changed; it's our context that's getting in the way.

How do we get around this?

## Taking Things Out of Context

When a value is wrapped-up in a context, applying a normal function to it just won't fly. What we need 
is a way to get at and act on our values inside their context. In the instance of our array example above, 
we can in fact do this very easily by mapping our `addOne` function over the array using [Lodash's `map`](http://lodash.com/docs#map):

{% highlight javascript %}
_.map([5, 6, 7], addOne);
//=> [6, 7, 8]
{% endhighlight %}

This probably looks very familiar, but what's actually going on under the hood? 

What `_.map` is doing here is what's called *lifting*; we *lift* our `addOne` into the array, call it using the value at each array index 
and then close the array back up:

{% highlight javascript %}
_.map([5, 6, 7], addOne);
//=> [addOne(5), addOne(6), addOne(7)]
{% endhighlight %}

`_.map` is taking the value *out of its context* to act on it, then boxing it back up into it's original context for us afterwards.

So `_.map` knows how to apply functions to values that are wrapped in an array. The question is: is
it more abstract than this? Can we apply it to other data types rather than just arrays? 

We certainly can; but there's just one caveat.

Underscore/Lodash's `map` function becomes next-to-useless when implementing our own interface. 
We need more flexibility, allowing us to define our own `map` functions for each new object we create.
From now on, we're going to be using [typeclasses](https://github.com/loop-recur/typeclasses), 
a small library that will make our lives much easier in this task.

## Functors

Ok, time to get serious. Deep breath and hold on to your butts.

Let's start vanilla by creating our own constructor object, giving us a new container for our values:

{% highlight javascript %}
var MyObj = function(val){
  this.val = val;
};
{% endhighlight %}

This is actually a good representation of how we treat types a little differently 
in functional programming. We're using objects as contexts or containers for values.

What we want to be able to do is map over this object and have a function run on it's contents. What 
we're aiming for is an interface that will perform the same operation of lifting our function into 
our object and acting on the values, just like we had above with `_.map`:

{% highlight javascript %}
// Mapping over MyObj...
map(addOne, MyObj(5));

// ...should look like this
MyObj(addOne(5));
{% endhighlight %}

What we need to do is define our own `fmap` on `MyObj`. We need: a **Functor**. Don't worry if you're not 100% certain what's 
going on here, we'll get right to it in a minute:

{% highlight javascript %}
Functor(MyObj, {
  fmap: function(f){
    return MyObj(f(this.val));
  }
};

fmap(addOne, MyObj(5));
//=> 6
{% endhighlight %}

What we've just done is define a **Functor**. A Functor is a [typeclass](http://learnyouahaskell.com/types-and-typeclasses#typeclasses-101), a sort 
of interface that defines a behaviour. If we want to get all Haskell up in here, this is the definition:

{% highlight haskell %}
class Functor f where
  fmap :: (a -> b) -> fa -> fb
{% endhighlight %}

All we're saying here is that to make a Functor from any data type, we just need to define how `fmap` will work with it. 

Congratulations! Now you know what a Functor is, we can talk about how we can put them to good use.

## Use You a Functor for Great Good!

So I'm sure the question on your lips right now is: how is any of this mumbo-jumbo useful? Turns out there 
are a number of practical applications for functors. Let's start with one we'll call `Maybe`:

### The Maybe Functor

{% highlight javascript %}
var Maybe = Constructor(function(val){
  this.val = val;
});

Functor(Maybe, {
  fmap: function(f) {
    return this.val ? Maybe(f(this.val)) : Maybe(null);
  }
});

fmap(addOne, Maybe(3));
//=> 4

fmap(addOne, Maybe(null));
//=> null
{% endhighlight %}

`Maybe` says that whatever context our function is mapping over might contain a value or it might not. 
If it does contain a value then run the function against it, but if it doesn't then don't run the function at all and return the `Maybe` back. 
This is a really handy little functor as it's basically an abstracted `null` check that we can map over!

Here's a real-world example. We're writing a user settings page for a client-side application. 
We have data for the currently logged-in user available to us, which we're using
to populate our settings page. However, our user felt a bit dubious about providing us with his 
address when he signed up for his account, meaning we have no data object for it. How do we deal with 
this when everything gets passed to our page for rendering?

This is where our friend `Maybe` can help:

{% highlight javascript %}
// Grab current user data
var currentUser = App.get(current_user);

// Before using our Maybe functor, we might be passing 
// a non-existent value to updateField causing our 
// whole app to blow up and barf an error at us
updateField(currentUser.address);

// Or, we can map our updateField function over
// our Maybe-wrapped value, giving us a null check 
// on our value! Not to mention a spot of 
// dynamic type-checking, too.
fmap(updateField, Maybe(currentUser.address));
{% endhighlight %}

...................................

## The Either Functor

Here's another functor, `Either`:

{% highlight javascript %}
var Either = Constructor(function(left, right){
  this.left = left;
  this.right = right;
};

Functor(Either, {
  fmap: function(f){
    return this.right ? 
           Either(this.left, f(this.right)) :
           Either(f(this.left), this.right);
  }
});

fmap(addOne, Either(5, 6));
//=> Either(5, 7);

fmap(addOne, Either(1, null));
//=> Either(2, null);
{% endhighlight %}

`Either` is an abstraction that provides you with defaults. When we try to map `addOne` over our `Either`, it will apply our function 
to the right value if it's present, or against the left value if it isn't.

How is this useful? Using our previous example, instead of just checking the existance of our value we can use our `Either` functor to 
provide our app with a default value. That way, our address field wouldn't be blank in the absence of data, but be
pre-filled with some default instead:

{% highlight javascript %}
fmap(updateField, 
     Either({ address: "The Post-Apocalyptic Land of Ooo" }, currentUser.address);
{% endhighlight %}

...................................

## Conclusion
Functors aren't these mystical programming entities resigned to exist only in the brains of Computer Science 
graduates, ivory-tower academics and Haskell. They are extremely useful constructs that make operating on data 
in a functional way much easier. Hopefully this post has given you some insight into how to take your 
JavaScript to the next level: use you a Functor!

## References
* [Functors, Applicatives and Monads in Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)
* ["Hey Underscore, You're Doing It Wrong!"](https://www.youtube.com/watch?v=m3svKOdZijA) - a talk by Brian Lonsdorf at [HTML5DevConf 2013](http://html5devconf.com/)
