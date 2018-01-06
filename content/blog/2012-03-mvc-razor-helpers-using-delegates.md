+++
aliases = ["/development/2012/03/09/mvc-razor-helpers-using-delegates/"]
date    = "2012-03-09T14:57:32-05:00"
tags    = [".net", "mvc"]
title   = "MVC Razor Helpers using Delegates"
+++

Once in a while, I find myself implementing the same helper functions in
different pages. In an effort to not repeat myself, I generally move helper
functions into a common cshtml file in App_Code (as
[suggested here](http://blog.slaks.net/2011/03/dissecting-razor-part-8-static-helpers.html)).

While this works great for a single application, I’ve found myself copying
helpers from project to project.

So in order to avoid the copy-paste nightmare, I decided to see if I could abstract the basic functionality into an
extension method and then have application specific rendering. This concept is similar to writing templated server
controls in a web forms environment.

The first step is to create a new class library and add a static class to it. I called my class
`HtmlHelperExtensions.cs`

{{< gist pseudomuto 6335058 "HtmlHelperExtensions_shell.cs" >}}

One particular helper I use a lot is rendering something based on a condition.  For example, show a logout button if the
user is logged in, or render items in there are any. The following methods can be used to accomplish this.

{{< gist pseudomuto 6335058 "HtmlHelperExtensions_methods.cs" >}}

Both of these methods are extensions for the `HtmlHelper` class (the one you’re using when you write `Html.SomeMethod()`
in your views). The first one takes a template, an `IEnumerable` of the type and a condition. The second is simply a
little syntactic sugar to allow passing a single item instead of an `IEnumerable`.

Here are some example calls for these functions.

{{< gist pseudomuto 6335058 "view.cshtml" >}}

Let’s assume you want to render something if the user is in a specific role.  You could accomplish this by adding some
more sugar.

{{< gist pseudomuto 6335058 "HtmlHelperExtensions_sugar.cs" >}}

Hopefully someone will find this of some use. Let me know what you think!
