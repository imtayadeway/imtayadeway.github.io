---
layout: post
title: "post the first"
date: 2014-04-20 21:53:00
---

This is the first post. It contains some text and some Ruby.  This is the text.  And here is the Ruby:

{% highlight ruby %}
# a comment

class SomeClass < SomeModule
  include Enumerable

  attr_accessor :some_accessor

  def some_method
    @instance_method = "a string #{ with.interpolation }"
    puts '<-- a keyword'
  end

  def iterator
    some_collection.each { |variable| puts variable }
  end
end
{% endhighlight %}