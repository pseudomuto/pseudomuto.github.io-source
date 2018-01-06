+++
aliases = ["/development/2013/07/28/testing-emberjs-apps-with-rails-and-qunit/"]
date    = "2013-07-28T04:31:04-05:00"
tags    = ["ruby", "javascript", "testing"]
title   = "Testing EmberJS Apps with Rails and QUnit"
+++

I recently started playing around with [EmberJS](http://emberjs.com/). While I was insanely impressed with how easy it
was to get a single-page app up and running, I was having difficulty getting [QUnit](http://qunitjs.com/) setup
correctly.

After some finagling I was able to get it working. In my particular case I'm using Ruby 2.0.0 and Rails 4.0.0, though
I'm sure this same setup would work with previous versions with minimal (if any) tweaks.

_**You will need to have Ruby/Rails, Node and CoffeeScript installed**_

## Creating the App

Open up a terminal and cd to your folder of choice (rails will create a new folder when you run these commands)

{{< gist pseudomuto 6334645 "make_app.sh" >}}

We now need to add the following to our `Gemfile`:

{{< gist pseudomuto 6334645 "Gemfile" >}}

And add the following to our application config:

{{< gist pseudomuto 6334645 "app_config.rb" >}}

Ember only has the testing helpers when in development mode, so we set the default to `:development` (in application.rb)
and override that value when in production.

Finally, we bootstrap/install EmberJS and QUnit:

{{< gist pseudomuto 6334645 "setup.sh" >}}

The `--head` parameter tells rails to pull the latest version of ember and ember-data. You can re-run this command at
any point to get the latest version.

The `-c` parameter for `qunit:install` tells qunit to use coffeescript (my preference but can be omitted if desired).

## Setting up the Test Environment

In order to have QUnit work with ember, we need to add the following code to our `test_helper.coffee` file (found in
`test/javascripts/`):

{{< gist pseudomuto 6334645 "test_helper.coffee" >}}

This creates an element for containing your app, adds the test helpers to `EmberApp` and adds a globally accessible
function called `exists` that can be used in your tests.

## Creating and Running Tests

Let's add a test for the default (application) template. To start, create a new file in `test/javascripts/integration/`
called `application_tests.js.coffee` and add the following code to it:

{{< gist pseudomuto 6334645 "application_tests.coffee" >}}

This code creates a simple module (shared by all tests in this file) and adds a simple test. I realize this test is very
simplistic, but the goal of the post is to explain the setup so I'm just using a contrived example that checks to see
that our main template is loaded.

### Running the Tests

At this point, we should run our test to make sure that it fails ('cause we're Red, Green, Refactor folks). Running the
tests is easy. Start your server.

{{< highlight bash >}}
$ rails s
{{< / highlight >}}

Now open a browser and go to <http://localhost:3000/qunit> (I'm assuming the default rails port here...adjust as
necessary). You should see a failing test now.

### Making the Test Pass

The last step in getting this working is to add the code needed to make the test pass. To do this we need to create a
template in `app/assets/javascripts/templates/` called `application.hbs` and add the following code to it:

{{< gist pseudomuto 6334645 "application.hbs" >}}

_The .hbs extension is used for handlebars. If you're using Sublime Text 2, there is a package aptly named "Handlebars"
that will handle syntax highlighting, etc for you._

If you refresh the test page, you should see that your test now passes.

## Celebrate!

In a future post, I'll cover how we can run these tests from the command line without needing to start web brick. If you
know of a good post already, add a comment and save me the time! :)
