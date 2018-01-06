+++
aliases = ["/development/2012/02/27/efficient-thread-safe-caching-with-asp-net/"]
date    = "2012-02-27T18:55:34-05:00"
tags    = [".net", "caching", "threading"]
title   = "Efficient, Thread-Safe Caching with ASP.NET"
+++

Let me start by saying that I’ve made this mistake on many occasions in the past. I just assume that static/shared objects are thread-safe. Things like `HttpContext.Current.Cache`...OOPS!

A typical implementation of a cache lookup might look like this:

{{< gist pseudomuto 6335102 "naive_cache.cs" >}}

Let’s analyze this code in a little detail. Here are the steps performed:

1. Load the object from the Cache
1. If the object is null (not found)
  * Create the object and store it in the cache
1. Return the object back to the caller

To start, we should ask, what happens if 10 simultaneous requests come through attempting to get to the cache object?

You guessed it, we will be inside that if statement 10 times! Typically we cache requests/objects that are expensive. Processing 10 times in a row could potentially (and often does) cause some serious performance issues.

As a first stab at fixing this, I’ve often seen people just throw a `lock` around the `Cache` object before they insert, like this:

{{< gist pseudomuto 6335102 "locked_cache.cs" >}}

While this is better, we still have an issue. If 5 threads are waiting because of the lock, they will all still run the creation code. This is because we are not checking the status of the object again within the if statement ([more info here](http://en.wikipedia.org/wiki/Double-checked_locking)). 

This is a quick one to resolve:

{{< gist pseudomuto 6335102 "if_lock_if_cache.cs" >}}

There is still a problem with this code. Do you see it?

Assume we have a web app that serves hundreds of requests per second. Locking is expensive. If we lock the whole `Cache` object, we will be adding a bottleneck to the whole system (that uses cached objects).

The (my) proposed solution to solving this is to make a class to handle dealing with the cache. This object will have a dictionary of lock objects (one for each cache key).

When we want to get an object from the cache, we follow these steps:

1. Load the object from the Cacge
1. If it's null, lock on the object specific to the cache key
  * Check again to be sure we need to create it
  * If so, create the object and add it to the cache
  * Release the lock
1. Returns the cached item

Here is the code I've started using. It should be noted, this only works if you **ALWAYS** use this class to retrieve cache objects. This will not protect against simultaneous access using this method and `HttpContext.Current.Cache` in the same application. In short, choose one method (I suggest the one below) and use it everywhere you access the cache.

{{< gist pseudomuto 6335102 "CacheManager.cs" >}}

An example call to this new method looks like this:

{{< gist pseudomuto 6335102 "ClientCode.cs" >}}

The create method is only called if necessary.

Hope this helps someone!
