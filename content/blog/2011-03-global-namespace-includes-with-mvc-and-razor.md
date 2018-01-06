+++
aliases = ["/development/2011/03/07/global-namespace-includes-with-mvc3-and-razor/"]
date    = "2011-03-07T00:00:00-05:00"
tags    = [".net", "mvc", "razor"]
title   = "Global Namespace Includes with MVC and Razor"
+++

While I'm loving MVC3 and the Razor view engine, I was sorely disappointed when I couldn't use the
`system.web/pages/namespaces` collection in `web.config` to add global namespace includes.

After a bit of hunting and looking through the web.config files underneath the views folder (in each Area), I found that
this can be accomplished by adding the following to the root `web.config` file:

{{< gist pseudomuto 6335507 >}}

Now you can add any namespaces you require under the new razor namespaces section.
