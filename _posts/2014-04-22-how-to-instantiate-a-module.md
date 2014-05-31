---
layout: post
title: "how to instantiate a module"
date: 2014-04-22 19:11:00
---

probably the most frequent basic ruby interview/trivia question: what's the difference between a module and a class? most frequent answer: you can't instantiate a module!

so why, after all, can't you instantiate a module? just for fun, can we fool a module into thinking it can instantiate an object for us? probably!

{% highlight ruby %}
module MessedUpModule
  def self.new(*args, &block)
    instance = Module.new
    instance.extend(self)
    instance.send(:initialize, *args, &block)
    instance
  end

  def class
    MessedUpModule
  end

  def to_s
    "#<#{ self.class }:0x#{ (object_id * 2).to_s(16) }>"
  end

  private

  def initialize(*args, &block)
    # etc..
  end
end
{% endhighlight %}

applications? well, none i can think of so far. but i enjoyed the exercise. yes, it's a kind of cheat, but then part of it was deciding how much of a cheat was acceptable. a few things were learned along the way and i thought deeper about ruby's object heirarchy.
