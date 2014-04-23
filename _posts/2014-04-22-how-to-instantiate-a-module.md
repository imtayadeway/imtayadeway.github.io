---
layout: post
title: "how to instatiate a module, and other pointless endeavors"
date: 2014-04-22 19:11:00
---

probably the most frequent ruby interview/trivia question: what's the difference between a module and a class? most frequent answer: you can't instantiate a module!

anything that follows is usually a list of differences mainly in attitudes toward the two, as opposed to the features. a slightly more sophisticated answer might point out that they're probably more similar than we're used to thinking.

so why, after all, can't you instantiate a module? further, why do modules have 'instance' methods, eh?

just for fun, can we fool a module into thinking it can instantiate an object for us? probably. it is ruby after all. and so, with some jiggery pokery:

{% highlight ruby %}
module MessedUpModule
  def self.new(*args, &block)
    klass = Class.new
    klass.include(self)
    instance = klass.allocate
    instance.send(:initialize, *args, &block)
    instance
  end

  def to_s
    "#<#{ self.class }:0x#{ (object_id * 2).to_s(16) }>"
  end

  def class
    MessedUpModule
  end
end
{% endhighlight %}

let's give it a road test:

{% highlight ruby %}
class SteadyClass; end

mum = MessedUpModule.new
sc = SteadyClass.new

p [mum, sc] # => [#<MessedUpModule:0x8c83590>, <SteadyClass:0x8c834b4>]

p [mum.class, sc.class]             # => [MessedUpModule, SteadyClass]
p [mum.class.class, sc.class.class] # => [Module, Class]

p mum.public_methods - sc.public_methods # => []
p sc.public_methods - mum.public_methods # => []

p mum.private_methods - sc.private_methods # => []
p sc.private_methods - mum.private_methods # => []
{% endhighlight %}

applications? none i can think of so far. but do let me know.