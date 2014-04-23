---
layout: post
title: "how to instatiate a module, and other pointless endeavors"
date: 2014-04-22 19:11:00
---

probably the most frequent ruby interview/trivia question: what's the difference between a module and a class? most frequent answer: you can't instantiate a module!

anything more elaborate than that usually takes the form of a list of differences in mostly attitudes toward the two, and not so much of the features. a slightly more sophisticated answer might point out that they're probably more similar than we're used to thinking.

so why, after all, can't you instantiate a module? further, why do modules have 'instance' methods, eh?

can we fool a module into instantiating an object for us? well, it ruby after all, so i'm going to say yes, with some jiggery pokery. here's a go:

{% highlight ruby %}
module MessedUpModule
  def self.new(*args, &block)
    klass = Class.new
    klass.include(self)
    klass.new(*args, &block)
  end

  attr_accessor :a, :b

  def to_s
    "#<#{ self.class }:0x#{ (object_id * 2).to_s(16) }>"
  end

  def class
    MessedUpModule
  end

  private

  def initialize(a, b)
    @a = a
    @b = b
  end
end

class SteadyClass
end

maya = MessedUpModule.new('foo', 'bar')
puts maya   # => #<MessedUpModule:0x8c83590>
puts maya.a # => foo
puts maya.b # => bar

yama = SteadyClass.new
puts yama # => #<SteadyClass:0x8c834b4>

p [maya.class, yama.class]             # => [MessedUpModule, SteadyClass]
p [maya.class.class, yama.class.class] # => [Module, Class]

p maya.public_methods - yama.public_methods # => [:a, :a=, :b, :b=]
p yama.public_methods - maya.public_methods # => []

p maya.private_methods - yama.private_methods # => []
p yama.private_methods - maya.private_methods # => []
{% endhighlight %}


applications? none, whatsoever.
