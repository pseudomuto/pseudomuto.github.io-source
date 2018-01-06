+++
aliases = ["/development/walkthroughs/2013/08/13/continuous-integration-for-net-with-travis-ci-and-xunit/"]
date    = "2013-08-13T17:07:53-05:00"
tags    = [".net", "ci", "automation"]
title   = "Continuous Integration for .NET with Travis CI and xUnit"
+++

For this post, I'm going to go over getting a simple CI environment setup using [Travis CI](https://travis-ci.org/
"Travis CI") and [xUnit](http://xunit.codeplex.com/ "xUnit").

If you're one of those people that just wants to see the code, <a title="Travis CI Demo"
href="https://github.com/pseudomuto/travis-ci-demo">click here</a>!

Our goal is to have the following scenario: whenever a push happens (to any branch) on GitHub, a build of the app will
be triggered by Travis CI. If the build succeeds, it will run all of the unit tests for your app. If those all pass, the
build is successful, otherwise it is considered a failed build. The build process will make use of NuGet to restore all
packages prior to build.

I'm going to assume we're starting with a brand new GitHub repo. In my case, I created a new repo with C# as the chosen
language and MIT for the license. After cloning, I have the following directory structure:

{{< highlight text >}}
<path/to/app>/
  .gitignore
  LICENSE
  README.md
{{< / highlight >}}

Because the goal here is to illustrate the process rather than the app, We're going to make a really simple app. I would
strongly suggest setting up CI as early on as possible on your projects as well. That way, when build issues arise you
can solve them one at a time (way easier).

## Creating the Projects

Let's create the application. In Visual Studio...

* Create a new `Blank Solution` called `src` and save it in the root of your GitHub repo
* In the solution explorer, rename your solution to `CI Demo`
* Create two new class libraries: `CI.Demo` and `CI.Demo.Tests` - *make sure to use .NET 4.0 (not 4.5)*
* Delete Class1.cs from both projects

We need to use .NET 4 for all projects in the solution. This is due to mono's lack of support for .NET 4.5.

You should now have the following structure:

{{< highlight text >}}
<path/to/app>/
  ...
  ...
  src/
    CI.Demo/
    CI.Demo.Tests/
    CI Demo.sln
{{< / highlight >}}

## Making it Work Locally

### The CI.Demo.Test Project

* Add a reference to the `CI.Demo` project
* Install the following NuGet packages:
  * xunit
  * ShouldFluent

Now add a new class called `Feature.cs` with the following code:

{{< gist pseudomuto 6334382 "FeatureTest.cs" >}}

## The CI.Demo Project

Now that we've got a few tests, let's make it work. Add the `Feature.cs` class with the following implementation:

{{< gist pseudomuto 6334382 "Feature.cs" >}}

The last step here is to enable NuGet package restore. Right-click on the solution and select "Enable NuGet Package
Restore." This will create the `src/.nuget` folder and add a few files to it.

At this point, the solution should compile and the tests should pass. This would be a good time to commit. **Be sure
that the .nuget folder is committed to git**

## Adding xUnit Dependency Files

In order to run xUnit tests from the Travis CI server, we will need to supply it with the path to the console runner.

For this we need to download xunit from <http://xunit.codeplex.com/> and copy the following files into
`<path/to/app>/lib/xunit/`

* xunit.console.clr4.x86.exe
* xunit.console.clr4.x86.exe.config
* xunit.dll
* xunit.dll.tdnet
* xunit.extensions.dll
* xunit.runner.msbuild.dll
* xunit.runner.tdnet.dll
* xunit.runner.utility.dll


You should now have the following directory structure:

{{< highlight text >}}
<path/to/app>/
  ...
  src/
    ...
    ...
  lib/
    xunit/
      ...files we just added
{{< / highlight >}}

## Setting Up NuGet Package Restore for Mono

Because we're using mono (Travis uses mono), we need to add a DLL to the .nuget folder in order for the msbuild task to
complete successfully. We also need to handle a weird formatting issue with NuGet.targets. This work will be done in
`/path/to/app/src/.nuget/`.

### Adding Dependecies

Let's start by downloading the
[Microsoft.Build.dll](https://github.com/pseudomuto/travis-ci-demo/blob/master/src/.nuget/Microsoft.Build.dll?raw=true)
assembly and adding it to the `/path/to/app/src/.nuget/` directory.

### Creating Mono Target File

Now we need to make a mono-specific version of the NuGet.targets file. Make a copy of NuGet.targets and save it as
NuGet.mono.targets. We need to modiy the `RestoreCommand` value by removing the literal space in the -solutionDir
parameter.

Original: `-solutionDir "$(SolutionDir) "`

Updated: `-solutionDir "$(SolutionDir)"`

I know...this is silly right? Unfortunately the space is required on Windows (at least with my set up), but it can't be
there for 'nix systems. In the future, I intend to have the Travis script modify the existing file so we don't need
two...but this is simpler for now.

## The Travis YAML file

We need Travis to do the following whenever a build is triggered:

* Update aptitude sources and install the mono development runtime and compiler
* Update the root CA certificates (since NuGet uses SSL)
* Use our mono specific targets file
* Ensure xUnit console is executable
* Set ENV variable EnableNuGetPackageRestore=true
* Build the solution and run the tests

A lot right? It turns out this is pretty straight forward. Create a new file called `.travis.yml` (notice the leading
dot) in `/path/to/app/`.

Add the following to it:

{{< gist pseudomuto 6334382 "travis.yml" >}}

Now we commit our changes and push to GitHub

{{< highlight bash >}}
git add .
git commit -am "setting up for travis integration"
git push
{{< / highlight >}}

## Making GitHub and Travis Hold Hands

At this point, all we need to do it tell Travis CI about our repo. Super simple...

* Login to <http://travis-ci.org/>
* Go to your account page and sync your GitHub repos
* Turn on Travis for your new repo

Now that it's all setup, we're going to add the build status to the README.md file and push to GitHub. This will trigger
the build.

Add the following to your README.md file (replace pseudomuto/travis-ci-demo with your GitHub repo)

{{< gist pseudomuto 6334382 "README.md" >}}

## Final Steps

* Commit and push to your repo
* Watch your build run in Travis CI (this can take a few minutes to start)
* Verify that the status icon shows in GitHub
* Celebrate!!


I've put the complete source code for this on [GitHub](https://github.com/pseudomuto/travis-ci-demo "Travis CI Demo")
for reference. Feel free to fork and make changes...
