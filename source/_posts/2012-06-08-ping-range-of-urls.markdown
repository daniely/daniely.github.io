---
layout: post
title: "Ping Range of Urls"
date: 2012-06-08 22:45
comments: true
categories: [Ruby]
---

**tl;dr** Coming from a Microsoft web dev background I'm often surprised at how simple yet
productive ruby can be. An example of this is a short (1 line) ruby script I wrote to ping a list of urls and
display summary results.

## Background

I'll be honest, switching from the Microsoft stack to the rails stack was hard. But one 
thing that helped me feel productive early on was focusing on ruby first. 
Not only is ruby essential to becoming a good rails dev but the little wins you have 
with it along the way will boost your morale as you struggle through learning 
how all the various pieces work together.

A few months back I wanted to find out which VPN server had the best ping.
Since ruby/rails forces (encourages?) you to get familiar with the command line I already
knew there were commands at my disposal that I just needed to leverage.

## The Program
The basic command I wanted to run was `ping vpn.test.com`. In ruby, there are 
several ways to get a string to run as if it were a command
typed into your terminal shell. The simplest is probably backticks. So if you
put `` `pwd` `` in a ruby script it will output something along the lines of
`/Users/Daniel/src`.

So to run ping all you need to do is `` `ping vpn.test.com` ``.
But I didn't want to run the command on just one url, I wanted it on a range.

```
ping vpn-1.test.com
ping vpn-2.test.com
ping vpn-3.test.com

etc... about 20 urls like this
```

Using ruby we can generate the list of commands (string literals) like so:

```ruby
commands = []   # array of commands
(1..20).each do |num|
  commands << "ping vpn-#{num}.test.com"
end

# commands
["ping vpn-1.test.com",
"ping vpn-2.test.com",
"ping vpn-3.test.com", etc... ]
```

Now we have a small problem, the command we want to run is no longer a string
literal. But it's easy enough to get around that by wrapping the command with
`#{}`

We also want to loop through and run each command. Here's how to do both:

``` ruby
commands.each do |command|
  `#{command}`
end
```

By default the `ping` command will continue pinging a url until a user cancels the command. 
In order to ping one url and then move onto the next without any user intervention
we need to specify a set number of times to ping a url. For that just add the `-c` flag
along with number of times to ping. In your terminal, running `ping -c2 vpn-1.test.com`
will result in output like this:

```
PING www.vpn-1.test.com (74.125.131.99): 56 data bytes
64 bytes from 74.125.131.99: icmp_seq=0 ttl=46 time=409.803 ms
64 bytes from 74.125.131.99: icmp_seq=1 ttl=46 time=407.988 ms

--- www.vpn-1.test.com ping statistics ---
2 packets transmitted, 2 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 407.988/408.895/409.803/0.908 ms
```

Notice that the url was pinged twice because of `-c2`. To capture this output in ruby just assign
it to a variable.

``` ruby
output = `ping -c2 vpn-1.test.com`
```

All I'm really interested in though is the summary info on the last line. So
let's parse the output. In our `ping` results, each line ends with a newline `\n`.
We can ue the `split` command on a string to split it up into an array. So doing 
`output.split("\n")` will result in an array like:

``` ruby output split into an array

["PING www.l.vpn-1.test.com (74.125.131.99): 56 data bytes",
"64 bytes from 74.125.131.99: icmp_seq=0 ttl=46 time=409.803 ms",
"64 bytes from 74.125.131.99: icmp_seq=1 ttl=46 time=407.988 ms",
"",
"--- www.l.vpn-1.test.com ping statistics ---",
"2 packets transmitted, 2 packets received, 0.0% packet loss",
"round-trip min/avg/max/stddev = 407.988/408.895/409.803/0.908 ms"]
```

Then to get the last element in the array call `.last` on the array. The
full command to parse the output and get the summary info is:

``` ruby parse to find summary info
output_summary = output.split("\n").last
```

But actually, we don't even need the variable `output_summary`. We can just parse
on the value returned from the command directly. Like so:

``` ruby
`ping -c2 vpn-1.test.com`.split("\n").last
```

To output something from your ruby script to the terminal just use `puts`.

Now putting all the pieces together here is our final script:

``` ruby
(1..20).each do |num|
  command = "ping -c2 vpn-#{num}.test.com"
  puts `#{command}`.split("\n").last
end
```
That's it! The first time I saw how little code I had to write I was simply blown away.
To run it, save the script to a file (for example `ping_urls.rb` then run it with
`ruby ping_urls.rb`. Here's what it looks like when you run the script.

```
host: vpn-1.test.com round-trip min/avg/max/stddev = 354.237/355.450/356.662/1.213 ms
host: vpn-2.test.com round-trip min/avg/max/stddev = 348.192/348.192/348.192/0.000 ms
host: vpn-3.test.com round-trip min/avg/max/stddev = 345.407/350.691/355.975/5.284 ms
host: vpn-4.test.com round-trip min/avg/max/stddev = 343.635/344.315/344.995/0.680 ms
host: vpn-5.test.com round-trip min/avg/max/stddev = 352.627/353.346/354.064/0.718 ms

etc...
```

Just to show you how terse ruby can be you can write the whole thing in one 
line without even creating a file to run it from. If you have ruby installed 
you can run a command with the `-e` flag. So typing `ruby -e 'puts "hi"'` in your
terminal shell will output `"hi"`. The terse version of the ping script is:

``` ruby terse version run directly from terminal
ruby -e '(1..20)each { |num| `ping -c2 vpn-#{num}.test.com`.split("\n").last }'
```

Simple yet powerful, no?
