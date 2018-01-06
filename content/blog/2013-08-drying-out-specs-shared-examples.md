+++
aliases = ["/development/2013/08/24/drying-out-specs-shared-examples/"]
date    = "2013-08-24T17:07:53-05:00"
tags    = ["ruby", "testing", "rspec"]
title   = "Drying Out Specs With Shared Examples"
+++

I generally find that most developers are fairly aware of and willing to apply the DRY principal on their main codebase.
However, when I look through people's tests, I find that the old copy/paste habit has found it's way into their workflow
(again?).

_I'm using "their code" to save face here...I found this in my own code and started looking for ways to clean it up._

So, let's take a look at an example of how we can remove some duplication from our specs. This example will be very
basic to illustrate the point, but a quick google search for `rspec + "shared_examples_for"` should turn up some more
detailed examples if you're interested.

## Let's Create a Simple Class

We start with a few simple tests

{{< gist pseudomuto 6331289 "0_simple_class_spec.rb" >}}

Now we create a class to make those tests pass

{{< gist pseudomuto 6331289 "1_simple_class.rb" >}}

## Adding Validation

Now we want our class to validate that name and description are supplied. As good RGR devs, we start by adding the new
tests.

{{< gist pseudomuto 6331289 "2_simple_class_spec.rb" >}}

Now we see the tests fail, so far so good. There are number of ways to do this kind of validation, but ActiveModel seems
to handle this well, and really who wants to write a bunch of validation code? So let's use the ActiveModel::Validations
module.

We start by adding "activemodel" to our gem file (skip this step if you're using rails).

{{< gist pseudomuto 6331289 "3_gemfile" >}}

Now we add the code to make the tests pass

{{< gist pseudomuto 6331289 "4_simple_class.rb" >}}

## OK, Tests are Passing...

If we look at the specs for the #name and #description properties, we see that they're essentially the same. Knowing
that we might have more properties on this class, we can setup our future selves for success by removing this
duplication now.

_**Developers, when you see this, rather than adding a comment stating that you know it's a problem...fix it! – You and
your whole team will be happier if everyone just does this.**_

For this step, we're going to create a set of shared examples that can be applied to any describe/context block.

{{< gist pseudomuto 6331289 "5_simple_class_spec.rb" >}}

And there you have it. Running the tests again will show that the tests still pass. The
shared_examples_for/it_should_behave_like construct can be really useful for drying out your specs.

We've now removed some duplication. Where else could we do this?
