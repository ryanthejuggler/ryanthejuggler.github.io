---
layout: post
title: "Chef, Vagrant, and DigitalOcean"
categories: devops
date: 2014-03-26 19:00:00
---
For a while now I've been longing for a personal infrastructure---a Git server, a Jenkins server, and some form 
of automated deployment. I've struggled (_really_ struggled) with chef-server on and off for the past few months 
to no avail. However, today someone at work showed me Vagrant. It turns out that there's a way to launch 
DigitalOcean instances using Vagrant---hooray!

## Preparation

### Vagrant
Vagrant is used for setting up and provisioning virtual machines (VMs). It can work with Chef to automatically configure your VM right after it's created. The installation is really easy---just download the appropriate package from [vagrantup.com](http://vagrantup.com/downloads.html) and install it.

### VirtualBox (optional)
VirtualBox will allow me to quickly spin up and spin down VMs on my local machine without having to wait for DigitalOcean to spin up a VM for me. Only use this if you plan to do development on your local machine.

This was also really easy; packages are available from [virtualbox.org](https://www.virtualbox.org/wiki/Downloads).

### Chef
Chef is what will release me from my worries about whether a box is configured or not. After Vagrant creates a VM, Chef will go through and set up the VM so that I can run my application on it.

Installation is a one-liner on most Linux distros:
{% highlight bash %}
curl -L https://www.opscode.com/chef/install.sh | sudo bash
{% endhighlight %}

> ### DigitalOcean
> [DigitalOcean](http://digitalocean.com) is my favorite cloud hosting provider at the moment, and luckily, Vagrant supports the creation of DigitalOcean VMs with the [vagrant-digitalocean](https://github.com/smdahlen/vagrant-digitalocean) plugin. 

> In order to follow the rest of this tutorial as-is, you'll need a DigitalOcean account. If you don't have one and don't want one, though, you can still keep going; just leave out any DigitalOcean-specific instructions. I've indented all the DigitalOcean-specific instructions like this section so you can easily skip them if you want.

> Installing this is a one-liner as well:

> {% highlight bash %}
vagrant plugin install vagrant-digitalocean
{% endhighlight %}

> After you do this, install the `digital_ocean` box:

> {% highlight bash %}
vagrant box add digital_ocean https://github.com/smdahlen/vagrant-digitalocean/raw/master/box/digital_ocean.box
{% endhighlight %}

## Setup
(Yes, setup is separate from preparation!)

I've decided to keep my Vagrantfiles in my Chef repo. First, I have to create my Chef repo. 

Opscode provides us with a barebones Chef repo, so we'll start there:

{% highlight bash %}
cd
git clone https://github.com/opscode/chef-repo.git
cd chef-repo
{% endhighlight %}

I'm going to keep my Vagrantfiles in the `boxes` folder in my Chef repo. I name my hosts after MBTA stops; the one for this tutorial will be called `alewife`.

{% highlight bash %}
mkdir -p boxes/alewife
cd boxes/alewife
{% endhighlight %}

To start working on my Vagrantfile, I call

{%highlight bash%}
vagrant init
{%endhighlight%}

from my `alewife` folder.

One last thing---I'm assuming you have SSH keys on your machine. If you don't, run `ssh-keygen` to generate them. Make sure one of your public keys is uploaded to DigitalOcean so you can provision a box with your key already on it.

[This Gist](https://gist.github.com/dergachev/3866825) gives a pretty awesome walkthrough of how Vagrant works together with Chef to launch and provision a box. The rest of this tutorial is pretty much an amalgamation of [that Gist](https://gist.github.com/dergachev/3866825), [a DigitalOcean tutorial](https://www.digitalocean.com/community/articles/how-to-use-digitalocean-as-your-provider-in-vagrant-on-an-ubuntu-12-10), and [a tutorial by Adam Brett](http://adamcod.es/2013/01/15/vagrant-is-easy-chef-is-hard-part2.html).

## Using Chef to provision

Now, we just have to:

* Describe to Chef what we want our machine to look like
* Connect Vagrant to the DigitalOcean API
* Get Vagrant to use our Chef recipe
* Launch the VM

### Describe to Chef what we want our machine to look like
This part pretty much follows [the tutorial on adamcod.es](http://adamcod.es/2013/01/15/vagrant-is-easy-chef-is-hard-part2.html).

First, we'll download the cookbooks we want to use, as well as their dependencies. It's a bit annoying that (1) these dependencies don't resolve themselves and (2) we have to specify even dependencies we don't use (lookin' at you, `windows` and `homebrew`), but it's not a totally absurd number.

{% highlight bash %}
cd ~/chef-repo
git submodule add https://github.com/opscode-cookbooks/apt.git cookbooks/apt
git submodule add https://github.com/opscode-cookbooks/apache2.git cookbooks/apache2
git submodule add https://github.com/opscode-cookbooks/mysql.git cookbooks/mysql
git submodule add https://github.com/opscode-cookbooks/php.git cookbooks/php
git submodule add https://github.com/opscode-cookbooks/openssl.git cookbooks/openssl
git submodule add https://github.com/opscode-cookbooks/iptables.git cookbooks/iptables
git submodule add https://github.com/opscode-cookbooks/logrotate.git cookbooks/logrotate
git submodule add https://github.com/opscode-cookbooks/build-essential.git cookbooks/build-essential
git submodule add https://github.com/opscode-cookbooks/homebrew.git cookbooks/homebrew
git submodule add https://github.com/opscode-cookbooks/windows.git cookbooks/windows
git submodule add https://github.com/opscode-cookbooks/chef_handler.git cookbooks/chef_handler
git submodule add https://github.com/opscode-cookbooks/iis.git cookbooks/iis
git submodule add https://github.com/opscode-cookbooks/xml.git cookbooks/xml
git submodule add https://github.com/opscode-cookbooks/yum-epel.git cookbooks/yum-epel
git submodule add https://github.com/opscode-cookbooks/yum.git cookbooks/yum
{% endhighlight %}

Then, we'll define a "role". A role is just a type of server; in production, for instance, you might have load balancers, app servers, and database servers. For this, you'd define a "load-balancer" role, an "app-server" role, and a "database-server" role.

First though, we want to make sure we don't publish any credentials. If it's not already there, create the `.chef` folder in the root of your chef-repo and edit `.chef/credentials.rb` with the following:

{% highlight bash %}
MYSQLPASS = 'Ch33$yBread'
{% endhighlight %}

Now, to make sure it doesn't get checked in, edit `.gitignore` in the chef-repo root and add a line containing `.chef/credentials.rb` so all our secrets will not be under version control.

Again from the root of your chef-repo, create `roles/test-box.rb` in your favorite editor and add:

{% highlight bash %}
# Name of the role should match the name of the file
name "test-box"

load '../.chef/credentials.rb'

run_list(
    "recipe[apt]",
    "recipe[openssl]",
    "recipe[apache2]",
    "recipe[mysql]",
    "recipe[mysql::server]",
    "recipe[php]"
)

override_attributes(
    "mysql" => {
        "server_root_password" => MYSQLPASS,
        "server_repl_password" => MYSQLPASS,
        "server_debian_password" => MYSQLPASS
    }
)

{% endhighlight %}

The run list specifies which recipes to execute and in which order. The `load` statement will run the ruby script we placed in the `.chef` folder to populate the variables.

> ### Connect Vagrant to the DigitalOcean API
> Let's edit the Vagrantfile to give it access to DigitalOcean. Open up `boxes/alewife/Vagrantfile` in your editor.

> First of all, you have to change `config.vm.box = "base"` to `config.vm.box = "digital_ocean"` to specify that we're going to use the `digital_ocean` box. Now, log into your DigitalOcean account and click on "API" on the left side. If you don't already have an API key, regenerate it here. Keep this tab open; we're going to need it in a moment.

> We don't want our credentials in version control, though, so we're going to break out our credentials into a separate file. The `.chef` directory in the root of our chef-repo is typically not checked in, so I think that's as good a place as any. Edit `.chef/credentials.rb` and add the lines:

> {% highlight ruby %}
CLIENTID = '0123456789abcdef0123456789abcdef'
APIKEY   = '0123456789abcdef0123456789abcdef'
{% endhighlight %}

> obviously replacing the strings with the actual client ID and API key.

> Now, back at the top of your Vagrantfile, add:
> {%highlight ruby%}
load '../../.chef/credentials.rb'
{%endhighlight%}

> Under the `config.vm.box` line, add:

> {% highlight ruby %}
config.vm.hostname = 'alewife'
config.ssh.private_key_path = "~/.ssh/id_rsa"
config.vm.provider :digital_ocean do |provider|
  provider.client_id = CLIENTID
  provider.api_key = APIKEY
  provider.image = "Ubuntu 12.10 x64"
  provider.region = "New York 2"
end
{% endhighlight %}

> This will take the variables we defined in `credentials.rb` and pass them to DigitalOcean to create our VM.

> By default, Vagrant will create a new key called "Vagrant" and push it to DigitalOcean. If you want to use an existing key instead, add a line `provider.ssh_key_name = "mykeyname"` immediately below the `provider.region` line.

> Finally, in order to use the `digital_ocean` provider by default, we'll add a line
> {% highlight ruby %}
ENV['VAGRANT_DEFAULT_PROVIDER'] = 'digital_ocean'
{% endhighlight %}

### Get Vagrant to use our Chef recipe
In order to make sure our new VM has Chef installed, we'll use the `vagrant-omnibus` plugin.
{%highlight bash%}
vagrant plugin install vagrant-omnibus
{%endhighlight%}

Next, back in the Vagrantfile, directly under the `Vagrant.configure(...) do |config|` line, add:
{%highlight ruby%}
config.omnibus.chef_version = :latest
{%endhighlight%}

Just before the closing `end` line of the whole file, add:
{% highlight ruby %}
config.vm.provision :chef_solo do |chef|
  chef.cookbooks_path = ['../../cookbooks']
  chef.roles_path = ['../../roles']
  chef.add_role 'test-box'
end
{% endhighlight %}

### Launch the VM with our new configuration
At this point, you can bring up the VM with
{% highlight bash %}
vagrant up
{% endhighlight %}

to launch a VM.

> If you've been following the "DigitalOcean" sections, it'll launch it with your newly configured `digital_ocean` provider. It'll take a couple minutes, so don't get anxious if it doesn't return right away. You'll see `Creating a new droplet...` on the screen for a bit, then `Assigned IP address: xxx.xxx.xxx.xxx` for what'll feel like forever. Pay attention to the IP address and write it down; you'll need it in a minute.

> _Leave your computer._ I mean it. Go for a walk or something. Go buy some celebratory beer.

Once the launch has finished, type the IP address into your browser. You should see the Apache "**It works!**" greeting. If you SSH into your machine, you'll find that MySQL and PHP have already been installed. Crack open those beers you just bought (you got them, right?) because you've just configured your server with minimal effort. You can destroy and bring up your server all day, and it'll always be set up just the way you want it to be.

## Explore

That's all I've got for you. There are pretty good tutorials all over the internet. I've found them intimidating in the past, but now that I have my workflow set up I find it pretty easy to set up anything I want.

If you want to learn more, you can explore the links I've sprinkled throughout the lesson. I've collected them here for your convenience.

### References

* [Vagrant + Chef tutorial by Dergachev](https://gist.github.com/dergachev/3866825)
* [How to use DigitalOcean as your provider in Vagrant on an Ubuntu 12.10](https://www.digitalocean.com/community/articles/how-to-use-digitalocean-as-your-provider-in-vagrant-on-an-ubuntu-12-10)
* [Vagrant is easy; Chef is hard](http://adamcod.es/2013/01/15/vagrant-is-easy-chef-is-hard-part2.html)
