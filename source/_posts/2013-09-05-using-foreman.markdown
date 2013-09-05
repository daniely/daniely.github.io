---
layout: post
title: "Using Foreman"
date: 2013-09-05 20:24
comments: true
categories: [Ruby, Rails, Foreman]
---

## Intro

One of the Rails apps I'm working on depends on a lot of locally running
services - database, job queue, web services, just to name a few. When I
started on the project I was opening up a new iTerm2 tab for each service
but that got old real quick. [Foreman](https://github.com/ddollar/foreman)
is a great tool for dealing with situations like this.

For a basic intro to foreman [go here](http://blog.daviddollar.org/2011/05/06/introducing-foreman.html).

## Tips

Most examples of foreman show you how to run multiple processes from the
same folder but what if you need to run them from separate folders? My project
is setup like so:

```
|-- project_root
    |-- website (rails)
    |-- account_services (sinatra)
```

A typical `Procfile` looks like:

```
website: bundle exec rails server -p 3000
```

But my `Procfile` is in the project root and I need to run both a rails webserver
in `website` and a sinatra app in `account_services` so that wont work. You can
however do this:

```
website: sh -c 'cd website && bundle exec rails s -p 3000'
account_services: sh -c 'cd account_services && bundle exec rackup -p 3001'
```

Use the `sh` command to change into the appropriate directory and run the
appropriate command to get your process running.

You'll notice that I specify port 3000 even though that's the default in rails.
The reason
is because sometimes you might end up with processes that don't get shut down by
foreman. In those cases you can use `pgrep -f 3000 | xargs kill` to kill them off.
Specifying the port number makes it easy to find the process using `pgrep`.

## Advantages

You might be wondering if it's really worth the effort. After all, it's just one
rails app and one sinatra app. In reality, my project has about 8
process that need to be up and running:

```
1 rails app
3 sinatra services
1 database
1 caching
1 job queue
1 messaging system
```

So rather than opening up 8 different tabs and fiddling with the history to find
the right commands I just go to my project root and type `foreman start` and
everything is up and running.

Another advantage is that the logs show up in one terminal window. At first I
didn't like that but it turns out to be very useful. When you have 8 different
processes outputting information it's nice to be able to see them aggregated in
one location. Plus, if you had to search in 8 different tabs you'd also have to scroll
through each one - what a pain.

For more fine grained control you could always run the process in their own window.
That's what I do with the rails website. I like to see the output in a separate
window and I might need to restart it when changing config files or something.

Anyway, if you start moving away from those monolithic rails apps and start
splitting them up into services take a look at [foreman](https://github.com/ddollar/foreman)
