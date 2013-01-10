---
layout: post
title: How to install PostgreSQL's hstore to your default database template
comments: true
related: false
description: How to install PostgreSQL's hstore to your default database template
---

I've started using PostgreSQL's hstore extention and data type. There are a few good resources out there about how to get this running in Rails.

- [http://travisjeffery.com/b/2012/02/using-postgress-hstore-with-rails/](http://travisjeffery.com/b/2012/02/using-postgress-hstore-with-rails/)
- [https://github.com/softa/activerecord-postgres-hstore](https://github.com/softa/activerecord-postgres-hstore)

It was working well for development and production but when I ran my tests and reset my test database, I kept running into errors. Even though I created the extension in my migration, the schema file didn't have that recorded so it failed every time:

    PG::Error: ERROR: type "hstore" does not exist

The first article I referenced suggested I add hstore to the default template. My first attempts failed because the only sql files I could find threw errors when I tried to add them to the default db template. So here's what I did.

I created `hstore.sql` in the contrib directory. (Mine was at `/usr/local/Cellar/postgresql/9.2.1/share/postgresql/extension`)

    CREATE EXTENSION hstore;

Then I ran this command to add it to the template:

    psql -f /usr/local/Cellar/postgresql/9.2.1/share/postgresql/extension/hstore.sql -d template1

Now my `db:test:prepare` works great!