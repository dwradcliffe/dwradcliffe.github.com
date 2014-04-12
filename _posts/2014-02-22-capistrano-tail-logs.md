---
published: true
layout: post
comments: true
related: false
description: How to easily tail your app logs with capistrano, even on mutliple hosts.
title: Tail your app logs with capistrano
---

Recently I deployed a new app using capistrano, and I wanted to tail the app logs easily. My app was deployed to two nodes, which complicates things just a little bit. There are a few snippets out there which discribed how to setup a task to accomplish this. These worked but I wanted to improve on them. Here's my modified snippet:


```ruby
task :logs do
  trap('INT') { puts; exit 0; }
  last_host = nil
  run "tail -f #{shared_path}/log/*.log" do |channel, stream, data|
    puts "\n\033[34m==> #{channel[:host]}\033[0m" unless channel[:host] == last_host
    puts data
    last_host = channel[:host]
    break if stream == :err
  end
end
```

I just need to run `cap logs` to get a nicely foratted stream of the logs in real-time. Hope this helps someone.
