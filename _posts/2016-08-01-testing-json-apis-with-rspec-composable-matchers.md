---
layout: post
title: Testing JSON APIs with RSpec Composable Matchers
tags:
- testing
- ruby
date: 2016-08-01 15:03:08
---

Testing JSON structures with arbitarily deep nesting can be
hard. Fortunately RSpec comes with some lesser-known composable
matchers that not only make for some very readable expectations but
can be built up quite arbitrarily too, mirroring the structure of your
JSON. They can provide you with a single expectation on your response
body that is diffable and will give you a pretty decent report on what
failed.

While I don't necessarily recommend you test every aspect of your API
through full-stack request specs, you are probably going to have to
write a few of them, and they can be painful to write. Fortunately
RSpec offers a few ways to make your life easier.

First, though, I'd like to touch on a couple of other things I do when
writing request specs to get the best possible experience when working
with these slow, highly integrated tests.

### Order of expectations

Because request specs are expensive, you'll often want to combine a
few expectations into a single example if they are essentially testing
the same behavior. You'll commonly see expectations on the response
body, headers and status within a single test. If you do this,
however, it's important to bear in mind that the first expectation to
fail will short circuit the others by default. So you'll want to put
the expectations that provide the best feedback on what went wrong
first. I've found the expectation on the status to be least useful, so
always put this last. I'm usually most interested in the response
body, so I'll put that first.

### Using failure aggregation

One way to get around the expectation order problem is to use failure
aggregation, a feature first introduced in RSpec 3.3. Examples that
are configured to aggregate failures will execute all the expectations
and report on all the failures so you aren't stuck with just the
rather opaque "expected 200, got 500". You can enable this in a few
ways, including in the example itself:

```ruby
it "will report on both these expectations should they fail", aggregate_failures: true do
  expect(response.parsed_body).to eq("foo" => "bar")
  expect(response).to have_http_status(:ok)
end
```

Or in your RSpec configuration. Here's how to enable it for all your
API specs:

```ruby
# spec/rails_helper.rb

RSpec.configure do |c|
  c.define_derived_metadata(:file_path => %r{spec/api}) do |meta|
    meta[:aggregate_failures] = true
  end
end
```

### Using response.parsed_body

Since I've been testing APIs I've always written my own JSON parsing
helper. But in version 5.0.0.beta3 Rails added a method to the
response object to do this for you. You'll see me using
`response.parsed_body` throughout the examples below.

### Using RSpec composable matchers to test nested structures

I've outlined a few common scenarios below, indicating which matchers
to use when they come up.

#### Use `eq` when you want to verify everything

```ruby
expected = {
  "data" => [
    {
      "type" => "posts",
      "id" => "1",
      "attributes" => {
        "title" => "Post the first"
      },
      "links" => {
        "self" => "http://example.com/posts/1"
      }
    }
  ]
  "links" => {
    "self" => "http://example.com/posts",
    "next" => "http://example.com/posts?page[offset]=2",
    "last" => "http://example.com/posts?page[offset]=10"
  }
  "included" => [
    {
      "type" => "comments",
      "id" => "1",
      "attributes" => {
      "body" => "Comment the first"
      },
      "relationships" => {
        "author" => {
          "data" => { "type" => "people", "id" => "2" }
        }
      },
      "links" => {
        "self" => "http://example.com/comments/1"
      }
    }
  ]
}
expect(response.parsed_body).to eq(expected)
```

Not a composable matcher, but shown here to contrast with the examples
that follow. I typically don't want to use this - it can make for some
painfully long-winded tests. If I wanted to check every aspect of the
serialization, I'd probably want to write a unit test on the
serializer anyway. Most of the time I just want to check that a few
things are there in the response body.

<br />

#### Use `match` when you want to be more flexible

```ruby
expected = {
  "data" => kind_of(Array),
  "links" => kind_of(Hash),
  "included" => anything
}
expect(response.parsed_body).to match(expected)
```

`match` is a bit fuzzier than `eq`, but not as fuzzy as `include`
(below). `match` verifies that the expected values are not only
correct but also that they are sufficient - any superfluous attributes
will fail the above example.

Note that `match` allows us to start composing expectations out of
other matchers such as `kind_of` and `anything` (see below), something
we couldn't do with `eq`.

<br />

#### Use `include`/`a_hash_including` when you want to verify certain key/value pairs, but not all
```ruby
expected = {
  "data" => [
    a_hash_including(
      "attributes" => a_hash_including(
        "title" => "Post the first"
      )
    )
  ]
}
expect(response.parsed_body).to include(expected)
```

`include` is similar to `match` but doesn't care about superfluous
attributes. As we'll see, it's incredibly flexible and is my go-to
matcher for testing JSON APIs.

`a_hash_including` is just an alias for `include` added for
readability. It will probably make most sense to use `include` at the
top level, and `a_hash_including` for things inside it, as above.

<br />

#### Use `include`/`a_hash_including` when you want to verify certain keys are present

```ruby
expect(response.parsed_body).to include("links", "data", "included")
```

The `include` matcher will happily take a list of keys instead of
key/value pairs.

<br />

#### Use a hash literal when you want to verify everything at that level

```ruby
expected = {
  "data" => [
    {
      "type" => "posts",
      "id" => "1",
      "attributes" => {
        "title" => "Post the first"
      },
      "links" => {
        "self" => "http://example.com/posts/1"
      }
    }
  ]
}
expect(response.parsed_body).to include(expected)
```

Here we only care about the root node `"data"` since we are using the
`include` matcher, but want to verify everything explicitly under it.

<br />

#### Use `a_collection_containing_exactly` when you have an array, but can't determine the order of elements

```ruby
expected = {
  "data" => a_collection_containing_exactly(
    a_hash_including("id" => "1"),
    a_hash_including("id" => "2")
  )
}
expect(response.parsed_body).to include(expected)
```

<br />

#### Use `a_collection_including` when you have an array, but don't care about all the elements

```ruby
expected = {
  "data" => a_collection_including(
    a_hash_including("id" => "1"),
    a_hash_including("id" => "2")
  )
}
expect(response.parsed_body).to include(expected)
```

Guess what? `a_collection_including` is just another alias for the
incredibly flexible `include`, but can be used to indicate an array
for expressiveness.

<br />

#### Use an array literal when you care about the order of elements

```ruby
expected = {
  "data" => [
    a_hash_including("id" => "1"),
    a_hash_including("id" => "2")
  ]
}
expect(response.parsed_body).to include(expected)
```

<br />

#### Use `all` when you want to verify that each thing in a collection conforms to a certain structure

```ruby
expected = {
  "data" => all(a_hash_including("type" => "posts"))
}
expect(response.parsed_body).to include(expected)
```

Here we don't have to say how many elements `"data"` contains, but we
do want to make sure they all have some things in common.

<br />

#### Use `anything` when you don't care about some of the values, but do care about the keys

```ruby
expected = {
  "data" => [
    {
      "type" => "posts",
      "id" => "1",
      "attributes" => {
        "title" => "Post the first"
      },
      "links" => {
        "self" => "http://example.com/posts/1"
      }
    }
  ]
  "links" => anything,
  "included" => anything
}
expect(response.parsed_body).to match(expected)
```

<br />

#### Use `a_string_matching` when you want to verify part of a string value, but don't care about the rest

```ruby
expected = {
  "links" => a_hash_including(
    "self" => a_string_matching(%r{/posts})
  )
}
expect(response.parsed_body).to include(expected)
```

Yep, another alias for `include`.

<br />

#### Use `kind_of` if you care about the type, but not the content

```ruby
expected = {
  "data" => [
    a_hash_including(
      "id" => kind_of(String)
    )
  ]
}
expect(response.parsed_body).to include(expected)
```
<br />

That's about it! Composable matchers are one of my favorite things
about RSpec. I hope you will love them too!
