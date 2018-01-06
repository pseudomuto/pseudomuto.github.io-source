+++
aliases = ["/development/2014/03/01/dining-philosophers-in-c/"]
date    = "2014-03-01T17:03:00-05:00"
tags    = ["c", "algorithms", "threading"]
title   = "Dining Philosophers in C"
+++

In a recent bout of insanity, I thought it would be cool to play around with concurrency in pure C. Nothing crazy, maybe
controlling access to a shared resource and a semaphore or two for good measure.

Since I assumed this would be no easy feat in C, I deciced I'd start with a problem I knew. So I went with the dining
philosophers problem.

## Defining the Problem

There are five (can be adjusted) philosophers sitting around a round table. These philosophers spend their days thinking
and eating. Each day, the philosophers think, eat, think, eat, think, eat and for change...think.

They eat from an endless supply of spaghetti. These philosophers are weird though, because they eat with two forks...I
know...I know...savages.

Now the problem...there are only five forks available at any given moment. So at most, only two philosophers can eat at
the same time.

## Things to Consider

We need to ensure that access to the forks is limited. The basic idea is that a philosopher will think while waiting to
get a hold of the fork to his left and right. Then he'll eat and release the forks allowing the philosophers to his left
and right to eat.

Since the day is shared by all philosophers, we'll model each philosopher as a thread so they can run at "the same"
time.

To do this, we'll need to ensure we don't get caught in a deadlock. This can happen if each philosopher grabs the fork
to their left, preventing anyone from grabbing the fork to their right. If that happens each philosopher thread would
block waiting for the fork on the right.
 
## The Code

{{< gist pseudomuto 9307707 >}}

## Walking Through the Code

### The `params_t` Struct

POSIX threads take a single `void *` parameter. However, we need to pass in four values to each of the threads. We need
to pass in the position of the current philosopher at the table, the total number of philosophers, the semaphore for the
critical region and the semaphores for the forks. To make this possible, we create a simple struct called `params_t` to
wrap these values.

### The `main` Function

We begin by defining a few semaphores. We need one semaphore for each fork and one for controlling the critical region.
Then we setup and run all of the philosopher threads. Finally, we call `pthread_exit` to wait for all threads to
complete.

### The `initialize_semaphores` Function

We initialize the forks by setting their initial value to `1`. We could have used mutexes for these, but since they're
really just binary semaphores I thought I'd stick to one type of lock throughout.

Next, we initialize the lock semaphore. This will be used to wrap the calls to `eat` in a critical region. We know that
only two philosophers can eat at the same time (since there are 5 forks and 2 are required to eat). We could initialize
the lock to `2` and it would technically be correct.  However, I like to do the simplest thing to avoid deadlock. The
simplest thing is to not let _all_ philosophers grab forks. So instead of `2`, I initialized it to `4`; one less than
the number of philosophers.

### The `run_all_threads` Function

This function spawns the philosopher threads. It creates an instance of `params_t` to pass to each thread and calls
`pthread_create` for each of them. Each thread will call the `philosopher` function passing it's instance of `params_t`
as the argument.

### The `philosopher` Function

This is the meat and potatoes of the app. Here is where each philosopher lives out their day. The interesting part is
surrounding the call to `eat`.

First, we wait on the lock. This is the start of the critical region and the part that ensures that no more than four
philosophers are attempting to pick up a fork at any given time. Next we wait on the left and right forks. Since these
semaphores are initialized to `1`, we can be sure that we are the only one that's using either of them at the moment.
Knowing that, we `eat`.

Finally, we post to the forks indicating we're done eating and post to the lock to exit the critical region to allow
other philosophers to eat.

## Summary

With a little effort, we've got a working solution to the dining philosophers problem. I know this example is very
contrived, but using POSIX threads seems relatively easy. If you're interested in learning about _pthread_, check out
[this tutorial](https://computing.llnl.gov/tutorials/pthreads/).
