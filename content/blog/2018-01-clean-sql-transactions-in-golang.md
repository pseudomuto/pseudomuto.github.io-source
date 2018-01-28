+++
title = "Clean SQL Transactions in Golang"
date  = "2018-01-27T19:36:10-05:00"
tags  = ["golang"]
+++

> **TL;DR**: Working with db transactions in Go should be simple, so I made some utilities to help with that.
You can see the gist containing [all the code here](https://gist.github.com/pseudomuto/0900a7a3605470760579752fcf0fc2b7).

The [`database/sql`](https://golang.org/pkg/database/sql) package has everything you need in order to work with db
transactions for most vendors. Typically, you'll write something like this:

{{< gist pseudomuto 0900a7a3605470760579752fcf0fc2b7 "main_1.go" >}}

While this code works, there are a few of things I think could be better about it.

**Every transaction has to worry about the details**

The flexibility of handling everything yourself can be nice, but its also a burden sometimes. For example, if one
statement fails, the next move is to rollback, always. Similarly, I would be worried about forgetting to commit the
transaction at some point (wouldn't be the first time).

**This kind of code seems like it should be a primitive in your codebase**

When you deal with common error scenarios (timeouts, retries, etc.) it's generally a good practice to introduce some
primitives in your codebase to handle these.

For example, a function like `FetchJSON(url string, params ...interface{}) error` can be used to set open/request
timeouts and handle any application specific errors in the same way everywhere. Then you just call this method whenever
you need to fetch some JSON.

**Not all "errors" are covered**

In the example above, what happens if one of the calls panics? This might not be a problem, but depending on the
`IsolationLevel` for the transaction, failure to explictly rollback could be an issue.

A better solution would recover from the panic, rollback, and then re-panic.

## Adding WithTransaction

The first thing we'll want to do is create a `func` to wrap the transaction logic so we don't need to do this
everywhere. There are several ways to do this, but I generally use something like this.

**Transaction Helper**

{{< gist pseudomuto 0900a7a3605470760579752fcf0fc2b7 "transaction.go" >}}

**Updated code for main**

{{< gist pseudomuto 0900a7a3605470760579752fcf0fc2b7 "main_2.go" >}}

This is better (IMO), but as we copy/paste this around the codebase, there's an aweful lot of repetition still
happening.

Ideally, we'd be able to supply a set of commands to be run, and have them all take place in the same
transaction.

## Implementing a Pipeline

We can create the concept of a _pipeline_ here that will run statements one by one within a transaction and chain the
last insert id along with each call.

{{< gist pseudomuto 0900a7a3605470760579752fcf0fc2b7 "pipeline.go" >}}

**Updated code for main**

{{< gist pseudomuto 0900a7a3605470760579752fcf0fc2b7 "main_3.go" >}}

This seems much easier to read, and is certainly easy to replicate across your application. I'm definitely interested in
opinions, so please leave a comment if you've got one!
