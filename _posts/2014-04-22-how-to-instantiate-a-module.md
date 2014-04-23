---
layout: post
title: "how to instatiate a module, and other pointless endeavors"
date: 2014-04-22 19:11:00
---

probably the most frequent ruby interview/trivia question: what's the difference between a module and a class? most frequent answer: you can't instantiate a module!

anything more elaborate than that usually takes the form of a list of differences in mostly attitudes toward the two, and not so much of the features. a slightly more sophisticated answer might point out that they're probably more similar than we're used to thinking.

so why, after all, can't you instantiate a module? further, why do modules have 'instance' methods, eh?

can we fool a module into instantiating an object for us? well, it ruby after all, so i'm going to say yes, with some jiggery pokery. here's a go at it:

{% highlight ruby %}
module MessedUpModule
  def self.new(*args, &block)
    klass = Class.new
    klass.include(self)
    klass.new(*args, &block)
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

# and later.....

mum = MessedUpModule.new('foo', 'bar')
puts mum   # => #<MessedUpModule:0x8c83590>
puts mum.a # => foo
puts mum.b # => bar

sc = SteadyClass.new
puts sc # => #<SteadyClass:0x8c834b4>


{% endhighlight %}


applications? none, whatsoever.
