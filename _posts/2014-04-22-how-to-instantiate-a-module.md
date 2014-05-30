---
layout: post
title: "how to instantiate a module"
date: 2014-04-22 19:11:00
---

probably the most frequent ruby interview/trivia question: what's the difference between a module and a class? most frequent answer: you can't instantiate a module!

anything that follows is usually heavy on attitudes, a little light on features. some might point out that they're probably more similar than we're used to thinking.

so why, after all, can't you instantiate a module? further, why do modules have 'instance' methods, eh?

just for fun then, can we fool a module into thinking it can instantiate an object for us? probably. it is ruby after all. and so, with some jiggery pokery, a template for a messed up module:

{% highlight ruby %}
module MessedUpModule
  def self.new(*args, &block)
    instance = Module.new
    instance.extend(self)
    instance.send(:initialize, *args, &block)
    instance
  end

  def to_s
    "#<#{ self.class }:0x#{ (object_id * 2).to_s(16) }>"
  end

  private

  def initialize(*args, &block)
  end
end
{% endhighlight %}

applications? none i can think of so far.
