---
layout: post
title: Finding the pid file for mongodb
---

For a while now I've been trying to monitor my mongodb instance with monit and obviously needed the pid file location. I read through my init script and looked in the places the script said the pid file was supposed to be, but it just wasn't there.

Finally I found that it's stored in the data directory.

    /data/mongodb/mongod.lock
    
Adding a simple file link will fix this up nicely for me:

    ln -s /data/mongodb/mongod.lock /var/run/mongodb.pid
    

I hope this helps someone else!