+++
aliases = ["/development/2013/10/07/pushing-files-to-github-programmatically/"]
date    = "2013-10-07T17:07:53-05:00"
tags    = ["api", "github", "nuget"]
title   = "Pushing Files to GitHub Programmatically"
+++

Recently I had the opportunity to work with GitHub's API again. This time I was particularily interested in the [Repo
Contents API](https://developer.github.com/v3/repos/contents/). The goal was to be able to publish files from disk or
embedded resource (read: stream) directly to a GitHub repo without having to clone the repo somewhere and then issue a
`git push`.

The API seemed (as is always with case with the GitHub API) straightforward and had methods for creating and updating
files. So I got to work...

[*TL;DR* - show me the code/installation instructions]({{< relref
"2013-10-pushing-files-to-github-programmatically.md#tldr" >}})

In my particular case, I was trying to pull embedded resources out of an assembly, push them to GitHub and then trigger
our CI server to run a build and if successful, trigger a deployment.

To get started, I had to think of what would be ideal for me. I came to the conclusion that I needed:

* an arbitrary array of bytes
* the target owner/repo
* a repo-relative path for where the bytes should be stored (file name)

The goal being code that looked like this:

{{< gist pseudomuto 6876762 "result.cs" >}}

I needed to make sure that the app could both create and update a file, so I ended up with the following code:

{{< gist pseudomuto 6876762 "ContentService.cs" >}}

<a href="#" name="tldr"></a>
## The Result

After only a few hours I had a fully tested implementation that I was able to use. I figured that other people might
have the same need, so I packaged it up as a NuGet package.

{{< highlight powershell >}}
Install-Package GitHubPushLib
{{< / highlight >}}

If you're interested, check out the [code on GitHub](https://github.com/pseudomuto/githubpushlib). I've setup Travis CI
and would love to see some pull requests!

The repo contains a sample app (with instructions) you can use to play around with.
