---
layout: post
title: Using better_errors with Sublime Text 2
comments: true
related: false
description: How to setup better_errors to open the source in Sublime Text 2
---

I have been using [`better_errors`](https://github.com/charliesome/better_errors) to provide a better error page on a few Rails 3 projects. It provides more information and gives me a live REPL at the source of the error.

This week I realized that the file name is actually a link to open the source file in my editor.

![](/images/2013/better_errors_filename.png)

The default link uses the TextMate handler (`txmt://open?url=...`), but since I'm using Sublime Text 2 I wanted to change the url and setup a handler on my system for ST.

First, the system handler. I searched around and found this: [https://github.com/asuth/subl-handler](https://github.com/asuth/subl-handler). After downloading and following the instructions, it works perfectly. Now ST responds to any `subl://` urls.

Next, changing the URL on the error page. After some [digging](https://github.com/charliesome/better_errors/pull/34/files) I was able to find out how to change the url. It's pretty simple; just add this to `config/environments/development.rb`:

    BetterErrors.editor = :sublime

And it works! Clicking on the filename will open the file to the exactly line in ST.