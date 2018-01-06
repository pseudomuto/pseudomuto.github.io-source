+++
aliases = ["/development/2013/12/11/freezing-and-thawing-git-state/"]
date    = "2013-12-11T17:07:53-05:00"
tags    = ["git", "shell"]
title   = "Freezing and Thawing Git State"
+++

We do lightening/dev talks every week at work, where a few developers get up and talk about cool projects or anything
they think is interesting. One of the recent talks by [James MacAuley] was about git freeze and git thaw.

The basic idea is that git stash kinda sucks sometimes. To show that, let's run through an example.

## Example of Using Git Stash

{{< highlight bash >}}
mkdir demo_repo && cd demo_repo
git init
git commit -m "initial commit" --allow-empty
touch README.md
touch LICENSE
git add README.md
{{< /highlight >}}

Now if we run `git status`, we should see the following:

![git status](img/git_status_001.png)

If we now stash our changes and get the status, we'll see that LICENSE is still untracked.

{{< highlight bash >}}
git stash
git status
{{< /highlight >}}

![git status](img/git_status_002.png)

This is intentional. As a rule, git will not do anything with files it's not tracking. This comes in handy for things
like password files (arguably should be git ignored) and other files that don't belong in git.

However, there are times when you want to take the current state of your working directory and stash it. And when you
`pop` the stash you want everything to go back to the way it was.

Luckily, git will convert any executable file in your path to a custom command. All we need to do it call it
`git-<command>` and voila!

## Git Freeze and Git Thaw

The goal of git freeze is to create two commits: `WIP [STAGED]` and `WIP [UNSTANGED]` containing the staged and unstages
files respectively.

These scripts were developed by [James Macauley]. You can check out the original gist
[here](https://gist.github.com/jamesmacaulay/582757).

{{< gist pseudomuto 7922871 "git-freeze" >}}

Thaw reverses the process by resetting `HEAD` appropriately.

{{< gist pseudomuto 7922871 "git-thaw" >}}

Both scripts take care to only create/revert commits if they're needed.

## Making It Work

These scripts should be placed in a directory in your `$PATH`. _**Make sure to make them executable.**_

Going back to our `demo_repo` example from earlier, let's pop that stash.

{{< highlight bash >}}
git stash pop
{{< /highlight >}}

You should see this again:

![git status](img/git_status_001.png)

Now we can run `git freeze` to uh..._freeze_ the current state. And `git thaw` to thaw it out again.

[James MacAuley]: https://twitter.com/jamesmacaulay
