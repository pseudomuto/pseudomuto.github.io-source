+++
aliases = ["/walkthroughs/2011/12/24/set-up-a-subversion-server-in-ec2/"]
date    = "2011-12-24T20:21:47-05:00"
tags    = ["subversion", "source control", "dev ops"]
title   = "Set up a Subversion Server in EC2"
+++

I recently wanted to setup a subversion server. Initially I thought of using
Beanstalk since they handle pretty much everything for you for a very small
monthly fee. However, I thought I might set up my own server in order to learn
the process.

I found a few online resources that outlined the process, but none of them were
exactly what I needed, so I decided to write this post to go over the steps in
detail. Hopefully this helps!

The steps we are going to take:

* Create a KeyPair, Security Group and Elastic IP in AWS
* Create a new Micro instance (64-bit Linux) in AWS
* Connect to the new instance using SSH
* Install a few tools on our new instance
* Create directories for repositories and auth settings
* Configure apache for servicing SVN requests
* Try it out

## Getting Set Up in AWS

The first step is to login to the AWS Management Console.

* On the left side of the screen under Network and Security, select KeyPairs.
* Create a new Key Pair called `SVN` - this will create the key pair and initiate a download. Store this key in a safe place - i.e. do not lose it!

Next, click on Security Groups (under Network and Security)

* Click Create Security Group
* Call it Web (Linux) and set the description to Subversion and HTTP
* Once it's been created, select the new group and click the Inbound tab (bottom of the screen)
* Add three inbound rules, SSH (22), HTTP (80), HTTPS (443)
* Apply your changes

Ok, now we have the "firewall" rules setup and a KeyPair (used to connect via SSH) to connect to an instance. Now we need to create a new server instance in EC2.

* On the left menu under Instances, select instances
* From the toolbar select Launch Instance
* Choose Launch Classic Wizard and click continue
* From the QuickStart tab, choose Basic 64-bit Amazon Linux AMI
* On the next screen, set the Number of Instances to 1, the Instance Type to "Micro" and click Continue
* From the Advanced Instance Options screen, leave all the defaults and click continue
* Add any tags you want and click continue
* On the next screen, select the KeyPair we created earlier and click continue
* From the Configure Firewall screen, select the Security Group we created earlier
* Click next/continue until the instance is created

We now want to connect and Elastic IP to our instance (this must be done every time you start the instance...Amazon doesn't automate this at all).

* From the left menu under Network and Security, select Elastic IPs
* Click Allocate New Address from the toolbar and click Yes when prompted
* Right click on the new IP address and select "associate" from the menu
* Select your new instance from the list

The last step would be to create an A record on your DNS server for the new IP. For the purposes of this article I'm going to assume that you've done this step, however while going through the rest of this post, you can simply replace `<domain>` with your Elastic IP address supplied by Amazon and get the same results.

## Connecting to Your New Instance

Now that we've got the server all setup in AWS, we need to connect to our new server and install some applications that will make things a little easier to work with.

There are loads of SSH applications out there that will let you use SSH from Windows (my preference is Putty - and it's free). If you're connecting from a Mac or any 'nix system, SSH is already built-in. For this article, I'll assume we're using a Mac/Nix installation. If you're using Windows, [check out this post](http://www.powercram.com/2009/07/connecting-to-aws-ec2-instance-linux.html) on connecting to EC2 with Putty.

Connecting from a Console

1. If you're using a Mac, you will need to open a terminal (`Applications->Utilities->Terminal`)
1. If you're using Nix, I assume you know how to open a terminal window :)
1. cd to the directory where you saved the KeyPair file (.pem) we created ealier
1. Change the permissions of this file (to stop SSH from complaining about it): <code>sudo chmod 600 <keypair></code>
1. Type command: <code>ssh -i <keypair> ec2-user@<domain></code> where <keypair> is the KeyPair file we created ealier and <domain> is to host you pointed to the Elastic IP (can be an IP address if you like)
1. The first time you connect, you might get a warning about trusting a certificate...type or select yes when prompted

## You're In! Now what?

Now that we're connected to the instance, we need to install a few things. I should note, I'm a big fan of emacs, so I always install it. If you're not familiar with Unix editors, stick to pico to keep things simple. For the remainder of this article, I'll use the word editor to refer to your preferred editor of choice on Linux.

Run the following commands to install subversion and svnserve.

{{< highlight bash >}}
$ sudo yum install mod_dav_svn subversion
{{< / highlight >}}

Next we're going to create a few new folders:

{{< highlight bash >}}
$ sudo mkdir -p /data/svn/repos /data/svn/logins /data/svn/scripts
$ sudo chown -R apache.apache /data/svn
{{< / highlight >}}

Now we need to create a subversion account to use for connecting to a repository. To do this we are going to create a new auth file and the create a script to handle adding new users in the future. To do this, run the following commands:

{{< highlight bash >}}
$ cd /data/svn/logins
$ sudo htpasswd -cm svn-auth-conf
{{< / highlight >}}

We should now make the login file a little more secure by running `sudo chmod 600 svn-auth-conf`. Next, we're going to create a script called create_user.sh that we can use to add user's in the future. This will make sure that we don't forget the commands and also ensure that we don't accidentally include -c again which will create a new file (overriding anything existing) - bad.

Using your editor, create the following script (in `/data/svn/scripts`), save it to a file called create_user.sh and then run sudo chmod +x create_user.sh to add execute rights to the file.

{{< highlight bash >}}
#!/bin/bash
user_name="$1"
auth_file="../logins/svn-auth-conf"

if [ "$user_name" = " ]; then
    echo -n "Enter user name: "
    read user_name
fi

chmod 700 $auth_file
if [ $? -eq 0 ]; then
    htpasswd -m $auth_file $user_name
    chmod 600 $auth_file
fi

{{< / highlight >}}

We can use this script by passing in a username or allowing the script to prompt us for one:

{{< highlight bash >}}
$ sudo ./create_user.sh davidmuto
# or
$ sudo ./create_user.sh
{{< / highlight >}}

Now we need a script that allows us to create repositories. The reason we make this a script is to save us from retyping the same commands every time we create a repo (very prone to error). Here's a script that will create a repository and set all of the necessary permissions on the files for being served by apache.

{{< highlight bash >}}
#!/bin/bash
repo_name="$1"
path_to_repo="

if [ "$repo_name" = " ]; then
    echo -n "What should the repo be called?: "
    read repo_name
fi

path_to_repo="../repos/$repo_name"
svnadmin create $path_to_repo
if [ $? -eq 0 ]; then
    sudo chown -R apache.apache $path_to_repo
    sudo chcon -R -t httpd_sys_content_t $path_to_repo
    echo "Successfully create repo '$path_to_repo'"
else
    echo "Could not create repository $path_to_repo"
fi
{{< / highlight >}}

## The Final Step!

Our last step it to set up the apache config file for servicing subversion repo requests.

* Navigate to /etc/httpd/conf.g
* Using your editor open the subversion.conf file for editing (much be run using sudo, e.g. sudo emacs subversion.conf or sudo vi subversino.conf)
* Add the following Location element to the file

    {{< highlight bash >}}
    <Location />
      DAV svn
      SVNParentPath /data/svn/repos
      AuthType Basic
      AuthName "My Subversion Realm"
      AuthUserFile /data/svn/logins/svn-auth-conf
      Require valid-user
    </Location>
    {{< / highlight >}}

* Restart the apache service: sudo service httpd restart


## Testing it Out

Now let's run through the process of creating a new repo and connecting to it.

* Navigate to `/data/svn/scripts`
* Create a user (if needed): `sudo ./create_user.sh [user_name]`
* Create a new repo: `sudo ./create_repo.sh [repo_name]`
* Using your favourite subversion client, checkout your new repo at `http://<domain>/<repo_name>`;

I hope this posts helps someone in need. If you guys notice anything I've missed, please feel free to let me know and I'll update this post as needed.

**Happy Coding!**
