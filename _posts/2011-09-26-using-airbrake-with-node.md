---
layout: post
title: Using Airbrake with Node.js and Express
---

We've been building a new app using [Node.js](http://nodejs.org/) and the [Express](http://expressjs.com/) framework. We're also using [CoffeeScript](http://jashkenas.github.com/coffee-script/), so my examples will use that. Error notification is an important part of every app and we use [Airbrake](http://airbrakeapp.com/) in all our apps. So naturally we wanted to use Airbrake with this new Node.js app.

There's a great unofficial library for node.js but it wasn't quite ready to use with Express. Here's what I did to make it all work together. Also I've hooked up the deploy notifications with capistrano.

## Install packages

Add the airbrake node package. If you've got a package.json you can add

<pre><code>"airbrake": "0.2.3"</code></pre>

and then `npm install`.

Otherwise you can just do `npm install airbrake`.

Add the gems to your gemfile:

<pre><code>gem 'capistrano'
gem 'airbrake'
</code></pre>

And finally `bundle`.

## Add airbrake error handler middleware

The airbrake node package doesn't yet have a built-in middleware piece for Express/connect, so I built one. It's not perfect and I'm working on getting it included in the official package. For now you will need to copy this into the lib.

Add this to the end of `node_modules/airbrake/lib/airbrake.js`:

<pre><code>Airbrake.prototype.errorHandler = function() {
  var self = this;
  return function errorHandler(err, req, res, next){
    if (res.statusCode < 400) res.statusCode = 500;
    self.log('Airbrake: Uncaught exception, sending notification for:');
    self.log(err.stack);
    err.url = req.url;
    err.action = req.route.path;
    err.params = req.params;
    self.notify(err, function(notifyErr, url) {
      if (notifyErr) {
        self.log('Airbrake: Could not notify service.');
        self.log(notifyErr.stack);
      } else {
        self.log('Airbrake: Notified service: ' + url);
      }
      res.send("Error!");
    });
  };
};
</code></pre>

## Setup in Express

In our main app file, we have a couple configure blocks.

<pre><code>app.configure 'development', ->
  app.use express.errorHandler dumpExceptions: true, showStack: true
app.configure 'production', ->
  airbrake = require('airbrake').createClient('YOUR-API-KEY')
  app.use airbrake.errorHandler()
</code></pre>

Here we are using the default express error handler in development mode to get nice error screens, and in production we are using airbrake.

## Deploy notification with capistrano

To get this to work nicely, I used the official airbrake cap tasks and rolled my own rake task.

Add this to your `Capfile` or `deploy/config.rb`:

<pre><code>require 'airbrake/capistrano'
</code></pre>

This will give you the cap task:
<pre><code>$ cap -T
cap airbrake:deploy      # Notify Airbrake of the deployment
</code></pre>

This task will call the rake task with the proper ENV variables set.

In our `Rakefile`, we have the following task defined:

<pre><code>namespace :airbrake do
  desc "Notify Airbrake of a new deploy."
  task :deploy do
    system "coffee lib/deploy.coffee"
  end
end
</code></pre>

Now the real deploy notification uses the node airbrake library again. In `lib/deploy.coffee`:

<pre><code>airbrake = require('airbrake').createClient("YOUR-API-KEY")

deployment =
  rev: process.env['REVISION'],
  repo: process.env['REPO'],
  env: process.env['TO'],
  user: process.env['USER']
  
airbrake.trackDeployment deployment, (err, params) ->
  throw err if err?
  console.log('Tracked deployment of %s to %s', params.rev, params.env)
</code></pre>

That script will require the library, build the deployment data from the ENV variables, and send it off to Airbrake. To test you can run `cap airbrake:deploy`.

That's all!

I've got an example app built on github for reference: [github.com/dwradcliffe/airbrake-express-example](https://github.com/dwradcliffe/airbrake-express-example)

My fork of node-airbrake with the handler: [github.com/dwradcliffe/node-airbrake](https://github.com/dwradcliffe/node-airbrake)