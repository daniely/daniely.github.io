---
layout: post
title: "Pairing With Avdi"
date: 2012-06-12 17:54
comments: true
categories: [Ruby, Pair Programming]
---

## Intro

Back in 2012 [Avdi Grimm](http://devblog.avdi.org/) was offering to pair program
for free - all you had to do was ask. Avdi [no longer](http://devblog.avdi.org/2013/02/19/taking-a-break-from-pair-programming/)
offers this amazing service but I wanted to at least share about one of our pairing
sessions as I feel very lucky to have had the opportunity.

##  Spree Installer Bug

The first problem we tackled was a small bug in [Spree Commerce](https://github.com/spree/spree).
During installation it would sometimes incorrectly detect a successful imagemagick
installation.  I had already submitted
a [pull request](https://github.com/spree/spree/pull/1533) but I didn't know
how to go about writing a test for it.

In the end we decided not to write a real test but we did do a quick spike.
Without going into too much detail, the original
pull request runs a shell command then checks the exit code. If it runs without
errors we can assume imagemagick is correctly installed.

``` ruby
# shell command to check if imagemagick is installed correctly
%x(identify -version)

# check exit code
$? => #<Process::Status: pid 16493 exit 0>

# better way to check
$?.success? => true
```

First thing Avdi says is, "lets see if we can override the %x command". I just
stared at him blankly on our screenshare. Wut? How? Do we write a method like
`def %x(command)` or something? I tried, it doesn't work. So what to do?

Another common way to run shell commands from ruby is to use backticks.

```ruby
`pwd` => "/Users/daniel/code\n"
```

Somehow Avdi knew that the `%x` syntax uses backticks. It's [documented](http://ruby-doc.org/core-2.0/Kernel.html#method-i-60)
but I was still surprised he knew this. He had a hunch that we could override
backticks and in turn override `%x` and that's exactly what we did.

``` ruby
class Foo
  def `(s)
    puts 'hello'
  end
end
```

But after we wrote this bit I was stuck again. How the heck do you call that
method? ``Foo.new`('some command')`` ??? That wont work. The answer is to use
`instance_eval`.

``` ruby
o = Foo.new
o.instance_eval { `ls` }
# => 'hello'

# with the %x syntax
o.instance_eval { %x[ls] }

# exit code ok?
$?  # => 0
```

The other issue was figuring out whether or not we could override the `$?`
value. But turns out it's read only.

This was the point when Avdi realized that there's no reason
to use backticks or the `%x[]` way of making system calls because these
versions are mainly used for their return values. All we want to know is if the
command executed successfully and relying on
`$?` for that info is in a way reaching outside of our methods boundaries.

A simpler/better approach is to call the command and see if
it was successful or not. That's it. And we can do just that with `system`. Thus our
code refactored to this:

``` ruby
success = system('identify -version')
```

We replaced 7 lines of code with just that one call in a new
[pull request](https://github.com/spree/spree/pull/1660/files)

## Double Error Messages

Then we talked about the double error messages that show up when you have a form 'hint'
that's the same as the error message. I was asking Avdi to help me figure out
how to "correctly" change the way rails handles this. But it turns out the
"correct" solution is to not touch rails at all. Instead, change the markup.
This was solved by adding a css style to hide the hint when an error
group is present. That's it! simple!

## Finding Solutions

This was a valuable lesson I learned from Avdi. While he has the technical chops
to program his way around things that's not what he turns to first.
Instead, he looks to see if the solution being implemented is the right fit.
Often times it's not and there might be a simpler solution to the problem. We just
saw two examples of that.

## Getting Better

For the last part of our session I asked for advice on improving as a developer.
These are the tips he gave in no particular order:

* Pair program
* Read code. rack is a good codebase to look at
* Read books. Blogs are ok but books tend to have less errors
  * Eloquent Ruby, Ruby Best Practices, The Ruby Way, Ruby Quiz,
    ruby talk mailing list, anything written by James Gray,
    Peepcode Play by Play

## Thanks Avdi for the awesome pairing session!
