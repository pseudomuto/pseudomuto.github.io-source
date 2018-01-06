+++
aliases = ["/development/2012/12/17/application-templates-for-rails/"]
date    = "2012-12-17T19:29:43-05:00"
tags    = ["automation", "rails", "ruby"]
title   = "Application Templates for Rails"
+++

Frequently I find myself creating basic rails apps to demonstrate a new feature, or prototype something for a spike
test. Every time I created an app, I would run `rails new -T`, then add RSpec, Cucumber, Haml, possibly Bootstrap, etc.

After the 10<sup>th</sup> app in a month, I decided that there must be an easier way. Then I saw this post:
<http://everydayrails.com/2011/02/28/rails-3-application-templates.html>

I started reading up on *generators* and *application templates*, then I thought I'd write my own, to see how it works.
I pushed my template to [GitHub](https://github.com/pseudomuto/rails-template).

Currently the template:

* Switches the layout engine to `Haml`
* Generates `app/controllers/HomeController` and adds an `index` action
* Generates `app/views/home/index.html.haml`
* Adds `root to: 'home#index'` to `config/routes.rb`

During the creation of your app, the template will ask whether or not you want to use these features/gems:

* Add and install `RSpec`
* Add and install `Cucumber`
* Add and install `twitter-bootstrap-rails` (with less), using `therubyracer` (will fix later)
  * removes `/app/views/layouts/application.html.erb`
  * creates fluid layout at `/app/views/layouts/application.html.haml`
* Init and Initial Commit for Git
  * create `.gitignore` with common rails exclusions
  * `git init and git add '.'`
  * `git commit -a -m "Initial commit."`


[Check it out](https://github.com/pseudomuto/rails-template) and let me know what you think!
