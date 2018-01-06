+++
aliases = ["/development/2014/12/27/running-rails-inside-vagrant/"]
date    = "2014-12-27T17:03:00-05:00"
tags    = ["rails", "vagrant"]
title   = "Running Rails Inside Vagrant"
+++

I use vagrant (along with [chef], a [vagrant plugin] and my [dotfiles]) for my development environment.  I do this for a
number of reasons, but mostly, I find it's the easiest way to get everything installed and configured properly
regardless of which physical box I'm working on.

There are a number of issues I needed to address to get this all working. One of those, was getting a simple rails
server from inside the VM to be available from my host's browser.

# The Problem

Normally, when running `rails s` either WebBrick (or even better Thin) starts up and listens for requests at
`http://localhost:3000`. This is not an issue if you're working on your local machine, but when attempting to access
that server from outside the VM we run into an issue.

From inside the VM, `localhost` is bound to the private `127.0.0.1` address. This means that, even if you do some port
forwarding for vagrant, the web server won't respond to requests from your host machine.

You could solve this by passing `-b 0.0.0.0` to `rails s` when you run it, but IMO `rails s` should just work whether
I'm running locally or within a VM. To get around this we need to force rails to listen on all interfaces by default.

# Configure Vagrant to Use a Static IP

The first thing we want to do is configure our VM to use a static IP address. This will allow us to set up host file
entries on our host machines that point to the VM. The simplest way to do this is to set this value in your
`Vagrantfile`

{{< gist pseudomuto ae8810b6b47fb090f8a1 "Vagrantfile" >}}

For this to take effect, you'll need to restart your VM (via `vagrant reload` or however else you'd like).

# Add Hosts Entry

Now that we've got the VM using a static IP, we can add an entry to our hosts file:

{{< highlight bash >}}
# do this on your host machine
$ sudo sh -c "echo 192.168.211.39 my-personal-vm.com >> /etc/hosts"
{{< /highlight >}}

Be sure to use the same IP you used for the VM and replace `my-personal-vm.com` with whatever you like.

# Tell Rails Server to Listen on All IPs

The last step is getting rails to listen for requests on `0.0.0.0` (all network devices). This is unfortunately not as
simple as I'd like, but doable nonetheless.

To do this, we're going to monkey patch `Rails::Server` if we're running in the `dev` environment.  There are several
places we could put this code, I tend to add it to _config/boot.rb_.

{{< gist pseudomuto ae8810b6b47fb090f8a1 "boot.rb" >}}

All we're doing here is keeping a reference to the original `default_options` method and defining a new method that will
call the original with the `Host` param set to `0.0.0.0`.

# Testing It Out

From your guest machine, run `rails s`. You should see something like this (note the binding address):

![thin console](img/thin-console.png)

Now from your host machine, open a browser and go to `http://my-personal-vm.com:3000` and you'll see you site! (Replace
`my-personal-vm.com` with whatever you used when adding the host entry).

[chef]: https://github.com/pseudomuto/kitchen-sink
[vagrant plugin]: https://github.com/pseudomuto/vagrant-pseudomuto
[dotfiles]: https://github.com/pseudomuto/dotfiles
