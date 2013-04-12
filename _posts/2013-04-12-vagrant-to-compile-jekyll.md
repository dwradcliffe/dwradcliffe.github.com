---
layout: post
title: Using Vagrant to compile Jekyll
comments: true
related: false
description: How to use Vagrant to compile a jekyll website
---

So let's say that you want to build a simple website using [Jekyll](http://jekyllrb.com/) but you don't want to run the compile step on your computer for some reason. Maybe you are using Windows or don't have ruby or something like that. It's trivial to use [Vagrant](http://www.vagrantup.com/) to spin up a quick virtual machine to do the work for you.

To start you will need to install Vagrant and VirtualBox. (Yes, I know you can use other providers such as VMware Fusion, but VirtualBox is free and easy.)

Download Vagrant: [downloads.vagrantup.com](http://downloads.vagrantup.com)

Download VirtualBox: [www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads)

You will need a `Vagrantfile` in your project root to configure Vagrant. Here is a boiled down version with everything you will need:

    Vagrant::Config.run do |config|

      config.vm.box = "precise32"
      config.vm.box_url = "http://files.vagrantup.com/precise32.box"
      config.vm.forward_port 4000, 4000
      config.vm.provision :shell,
        :inline => "sudo apt-get -y install build-essential && sudo /opt/vagrant_ruby/bin/gem install jekyll rdiscount --no-ri --no-rdoc"

    end

Now you are ready to start your VM. From a command prompt in your project directory:

    $ vagrant up
    $ vagrant ssh

Now you can start Jekyll on the VM:

    $ cd /vagrant
    $ jekyll --auto --server 4000

Now you should be able to use your own regular web browser to view the site at [http://localhost:4000](http://localhost:4000)

When you're done testing, just exit jekyll with a `ctl+c` and exit the vm with `exit`. Finally you can `vagrant destroy` to clean up.