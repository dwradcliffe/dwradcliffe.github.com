---
published: true
layout: post
comments: true
related: false
description: Using Chef DK alongside rbenv for development
title: Using Chef DK alongside rbenv
---

Chef recently released the [Chef Development Kit](https://downloads.getchef.com/chef-dk/) to help developers get their environment configured for Chef easily. This package includes Chef, Berkshelf, Test Kitchen and more. The idea is that to work on a cookbook or chef environment you wouldn't need to `bundle install` or setup any other rubies/gems/dependencies to get started working. This is a great idea and probably the best side effect is that we don't need to type `bundle exec` anymore!

However, Chef DK works best when it's your primary and only ruby environment. This doesn't play well with rbenv for ruby version management. My job involves both Chef development and non-Chef development. I use multiple ruby versions and that won't be changing anytime soon. So I needed to find a way to make Chef DK work well with rbenv.

Here is the root of the problem: my `$PATH`. Chef DK places symlinks to it's core binaries in /usr/bin for easy access - which is great. But if you have any of the chef & co gems installed in any ruby version, rbenv will create a shim that comes first in your PATH, preventing access to the Chef DK binaries.

Example:

    $ /usr/bin/knife --version
    Chef: 11.14.6

    $ which knife
    /Users/david/.rbenv/shims/knife

    $ echo $PATH
    /Users/david/.rbenv/shims:/usr/local/bin:/usr/bin

So what if I move the `/usr/bin` to the front of my PATH? Then the system gems will take precedent over the rbenv gems. That won't work.

The chef binaries in `/usr/bin/` are just symlinks... so where do they link to?

    $ ls -l /usr/bin/knife
    lrwxr-xr-x  1 root  wheel  21 Sep  1 13:42 /usr/bin/knife -> /opt/chefdk/bin/knife

So all the binaries are really in `/opt/chefdk/bin/`. Let's add that to my PATH! Here is the relevenat snippet from my `~/.zshenv` file:

    eval "$(rbenv init -)"
    export PATH="/opt/chefdk/bin:$PATH"

The order here is important; we need to make sure rbenv is setup before adding chefdk to the front of my PATH.

Now everything works:

    $ echo $PATH
    /opt/chefdk/bin:/Users/david/.rbenv/shims:/usr/local/bin:/usr/bin

    $ which knife
    /opt/chefdk/bin/knife

    $ which gem
    /Users/david/.rbenv/shims/gem
