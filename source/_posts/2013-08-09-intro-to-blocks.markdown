---
layout: post
title: "Intro to Blocks"
date: 2013-08-09 17:30
comments: true
categories:
published: false
---

**tl;dr** A beginner's intro to ruby blocks

## Blocks

The first block I ever used was `each`.

```ruby
[1, 2, 3].each do |item|
  puts item
end

# output
1
2
3
```

While it was easy for me to
understand the results I got back while using it, it took awhile before I
really understood what was going on. In order to unlock my understanding of
how `each` worked, I really needed to understand blocks first.

In ruby, blocks are the bit of code between curly braces
`{ }` or `do` and `end` that come after a **method call**. That last part is
super important, anytime you see a
pair of `{ }` or `do end` you need to realize it's being passed to a method.

When we use `each` we're calling a method that belongs to `Array`. But there's
nothing stopping us from passing a block to any method we want. And
actually, doing so should help us understand blocks more.

``` ruby
# does not explicity accept blocks
def say_hi_method
  puts 'hi'
end

# block passed to method and perfectly valid
say_hi_method do
  puts "in a block"
end

# same as above
say_hi_method {
  puts "also in a block"
}
```

Any method (like `say_hi_method` for example) can implicity take a block as
a parameter. But it's not very interesting until you do something with the
block.

To interact with the block we use the `yield` keyword, which is used to
execute the block. A slightly more interesting example:

``` ruby
def say_before_after
  puts 'before'
  yield
  puts 'after'
end

say_before_after do
  puts 'yield called'
end

# output
'before'
'yield called'
'after'
```

Empty blocks are also possible (though useless) but usually we check if a 
block was passed in using `block_given?`

```ruby
def say_before_after
  puts 'before'
  yield if block_given?
  puts 'after'
end

say_before_after do
  # empty block...pretty useless but still possible
end

#output
'before'
'after'
```

`yield` and `block_given?` are an easy way to start using blocks in an implicit
way. But if you want more fine grained, explicit control, you can prepend
a parameter name with `&`. This will turn the parameter into a block which 
can be called using `call`.

```ruby
def say_before_after(&block)
  say 'before'
  block.call
  say 'after'
end

say_before_after do
  puts 'from block'
end

# output
'before'
'from block'
'after'
```

The `block_given?` equivalent in this case is just a check to see if `block`
exists.

```ruby
def say_before_after(&block)
  say 'before'
  block.call if block
  say 'after'
end
```

Of course there are rules to passing in blocks as parameters. There can only
be one and it must be the last parameter passed in.

```ruby
# valid
def valid_method_with_block(first_name, last_name, &my_block)
  ...
end

# invalid
def invalid_method_with_block(first_name, &my_block, last_name)
  ...
end

# also invalid
def invalid_method_with_two_blocks(first_name, &my_block, &another_block)
  ...
end
```

TODO:

Now to make it more interesting, blocks can also take parameters!


...

With all these pieces in place, now we can fully understand what's 
happening in `each`...or rather, we can implement our own version.
...


The epiphany for me was realizing that blocks are ALWAYS linked to a method
somewhere.  Realizing this helped me make sense of a lot of magical code I 
saw in the ruby world. Take rspec for example:

``` ruby
# rspec and it's magical desc, it blocks
describe "SomeClass" do
  it "does stuff" do
    ...
  end
end
```

I used rspec for over a year before I realized that `describe` and `it` are
just methods. Their method signatures might look something like:

``` ruby
def describe(description, &block)
  ...
  block.call if block
end

def it(description, &block)
  ...
  block.call if block
end
```

In almost all cases you can use either notation and they mean the same thing. To keep
things simple let's ignore the minute differences between them.

Think of code blocks as chunks of code. The interesting thing about blocks is
that you can keep track of them, pass them around, and call them. To call the
code block use the `yield` keyword.

## draft - write out what they mean
``` ruby
def takes_a_block
  puts 'before'
  yield
  puts 'after'
end

takes_a_block do
  puts 'from outside'
end

# before
# from outside
# after
```
When first learning about blocks I found it helpful to write out exactly what
was being created or called.

``` ruby
def takes_a_block()
  puts 'before'
  # yield
  puts 'from outside'
  puts 'after'
end

takes_a_block do
  puts 'from outside'
end
```

In trivial cases it might not be very helpful but let's look at a slightly more
complex case, one where `yield` takes a parameter.

``` ruby
def takes_a_block()
  puts 'before'
  yield "-from yield"
  puts 'after'
end

takes_a_block do |msg|
  puts 'from outside' + msg
end

# before
# from outside-from yeild
# end
```

The first time I saw yeild taking a parameter it would really confuse me. So 
again, I just wrote out the things that were replaced.

``` ruby
better example...
```

## Procs and Lambdas

A brief note on terminology, when rubyists talk about procs and lambdas they
are pretty much referring to something in programming known as closures.

Conceptually, procs are not that hard to understand - they
are just references to methods along with some context for any variables contained
within them.

```
# simple proc example
proc1 = Proc.new { a = 1; puts a }


```

Procs and lambdas can also be considered virtually the same thing.
Again, there are differences between the two but for simplicity let's ignore them.

There are two ways to execute the proc, `my_proc.call` and `my_proc[]`

The part that used to confuse me was when examples I saw would mix simple proc
definitions with calls to
I used to have a hard time understanding ruby procs until I started to write out
what they represented. An example should help clarify:

Let's write a program that will wrap html tags around some text. We'll do it a
hard coded way first and later revise it to use lambdas.

``` ruby
def wrap_with_bold(text)
  return "<b>" + text + "</b>"
end
```

First of all, this code needs to be refactored. Methods in ruby return the last
statement executed so no need for the `return`. Also, there are performance
issues with using `+` to concatenate strings so it's preferable to use string
interpolation. Here's our refactor:

``` ruby
def wrap_with_bold(text)
  "<b>#{text}</b>"
end
```

Now let's create a method for wrapping with h1 tags.

``` ruby
def wrap_with_h1(text)
  "<h1>#{text}</h1>"
end
```

For every new tag we want to add we'll have to add another method. So we can already see this is not a great implementation.
One way to fix this is by passing in the tag along with the text.

``` ruby
def wrap_with(tag, text)
  "#{tag}#{text}#{tag}"
end
```

That's slightly better but let's try it with lambdas.

``` ruby
def wrapper(tag)
  lambda do |text|
    puts tag + text + tag
  end
end
```

Now, this might not look all that special but check out what we can do.

``` ruby
bold_wrapper = wrapper("<b>")
h1_wrapper = wrapper("<h1>")
```

Using lambdas we just created two methods that are functionally equivalent of
our hard coded methods `wrap_with_bold()` and `wrap_with_h1()`. To make it
crystal clear, they're functionally equivalent to:

``` ruby
def bold_wrapper(text)
  "<b>#{text}<b>"
end

def h1_wrapper(text)
  "<h1>#{text}<h1>"
end
```
