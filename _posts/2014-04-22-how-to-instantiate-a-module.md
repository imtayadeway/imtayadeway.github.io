---
layout: post
title: "how to instantiate a module"
date: 2014-04-22 19:11:00
---

probably the most frequent basic ruby interview/trivia question:
what's the difference between a module and a class? most frequent
answer: you can't instantiate a module!

i wonder then if we can fool a module into thinking it can instantiate
itself for us?  probably!

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

as it turns out, it doesn't take much work get a basic instantiation
metaphor going.  i've started to ape some classical behavior in
`#to_s`, but the rest is left to the imagination. so i suppose the
next question might be, how much work would it take to create an
object that was not a class, but was indistinguishable from one? the
answer to that question is probably best left to another post.

applications? well, none i can think of so far (hence my not wishing
to pursue it any further). but i enjoyed the exercise: ruby is fun,
and lets you have fun without even needing a real purpose. take that,
java!
