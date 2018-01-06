+++
aliases = ["/development/2015/01/27/compiling-vim-with-ruby-support/"]
date    = "2015-01-27T17:03:00-05:00"
tags    = ["vim", "chef"]
title   = "Compiling Vim with Ruby and Python Support"
+++

Like a lot of developers, I use vim. Of course, I don't use straight out of the box vim though. I use Vundle and other
settings (see [my dot files] for more) to make it better.

Sometimes there are plugins, or packages that require vim to be compiled with support for ruby and/or python. One
example of this is the excellent [Command-T] plugin.  Unfortunately, the default package doesn't include these flags
during compilation. But fear not, the process is simple enough and only takes a few minutes.

## Downloading and Extracting the Source

If you're feeling adventurous, you could just clone the source and build straight from the master branch ([git mirror]).
However, I would strongly suggest grabbing a copy of the latest actual release.

To see a list of releases check out [the releases page here].

At the time of this writing, the latest version was `7-4-589`, so I'll assume that's the version you want. If you want a
different version, simply replace all instances of `7-4-589` with the desired version number.

To get started, let's download the release. If you're using a browser, just click on the `tar.gz` link for the release
on the [releases page].

If you use vagrant or simply don't have a browser available, you can grab the file using curl like this:

{{<highlight bash >}}
curl -L https://github.com/b4winckler/vim/archive/v7-4-589.tar.gz > vim.tar.gz
{{< /highlight >}}

Now we need to extract the tar file. feel free to use a GUI tool if you have one. I prefer the console personally.

{{< highlight bash >}}
tar -zxvf vim-7-4-589.tar.gz
{{< /highlight >}}

## Building and Installing Vim

Now we need to build and install vim using the `rubyinterp` and `pythoninterp` flags. This is fairly straightforward:

{{< highlight bash >}}
cd vim-7-4-589
./configure --enable-rubyinterp --enable-pythoninterp
make && sudo make install
{{< /highlight >}}

You can verify that it worked by running `vim --version`. You should see a `+` next to both ruby and python now. Like
this:

![vim version](img/vim-version.png)

## Automating With Chef

If you use chef to provision your dev environment ([like me]), you can use this recipe to automate what was described
above.

### Recipe

Be sure to add `depends "ark"` to your cookbook's _metadata.rb_ file.

{{< gist pseudomuto 973859c44a7d27eae009 "vim.rb" >}}

### Attributes

{{< gist pseudomuto 973859c44a7d27eae009 "attributes.rb" >}}

If you're not using `7-4-589` be sure to generate the correct hash like so:

{{< highlight bash >}}
shasum vim-<version>.tar.gz -a 256
{{< /highlight >}}

Happy Viming!

[my dot files]: https://github.com/pseudomuto/dotfiles
[Command-T]: https://github.com/wincent/Command-T
[git mirror]: https://github.com/b4winckler/vim
[the releases page here]: https://github.com/b4winckler/vim/releases
[releases page]: https://github.com/b4winckler/vim/releases
[like me]: https://github.com/pseudomuto/kitchen-sink
