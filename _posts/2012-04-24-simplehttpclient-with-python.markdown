---
layout: post
title: "SimpleHTTPServer with Python"
---

## Forget MAMP

Time was that to get a local HTTP client running for development, I would simply fall onto [MAMP](http://www.mamp.info) to get me up and running. It seemed like the most pragmatic choice at the time. Plus I was kinda lazy.

It didn't take long before I began to think that there must be an easier, simpler way than having to rely on yet another tool/program running in the background, especially with something so straightforward yet so integral to my workflow.

## Enter Python.

In Paul Irish's [presentation at jQuery UK 2012 recently](http://vimeo.com/40929961), there's this little nugget of wisdom on how to get a really simple localhost server running on your Mac, without relying on a program like MAMP to do it for you.

In the Terminal, just type out the following:

{% highlight bash linenos %}
# adds an alias of 'server' specifying port 8000 (which can be set to anything you like)
$ alias server='open http://localhost:8000 && python -m SimpleHTTPServer'

# navigate to the directory your developing in
$ cd path/to/app/

# run a localhost server on port 8000 and open your default browser 
$ server
{% endhighlight %}

If there's an `index.html` file in your directory, that file will be served to you initially.

Every Mac has Python and SimpleHTTPServer installed. 

For more information on SimpleHTTPServer, check out the [Python documentation](http://docs.python.org/library/simplehttpserver.html).
