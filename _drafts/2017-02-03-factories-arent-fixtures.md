---
layout: post
title: Factories aren't Fixtures
date: 2017-02-03 19:17:46
---

As someone who learned both to program and to test for the first time
with Rails, I was quickly exposed to a lot of opinions about testing
at once, with a lot of hand-waving. One of these was, as I remember
it, that Rails tests with fixtures by default, that fixtures are
problematic, that Factory Girl is a solution to those problems, so we
just use Factory Girl. I probably internalized this at the time as
"use Factory Girl to new up objects in tests" without really
questioning why.

Some years later now, I sincerely regret _not_ learning to use
fixtures first, to experience those pains for myself (or not), to find
out to what problem exactly Factory Girl was a solution. For, I've
come to discover, Factory Girl doesn't prevent you from having the
same issues that you'd fine with fixtures.

Another thing I regret is not simply using my own factory methods in
tests, again, before finding out what problem with _that_ Factory Girl
was trying to address.

Consider the following:

```ruby
it "has a name" do
  user = create_user(name: "Bob")
  expect(user.name).to eq("Bob")
end

def create_user(attributes)
  User.create!({name: "Alice", date_of_birth: 30.years.ago}.merge(attributes))
end
```

A user model exists which validates the presence of a name and a date
of birth. This factory method solves the problem of my not wanting to
specify every required attribute when creating a user, giving it some
sensible defaults, but allows me to override any of those defaults for
the purposes of the behavior I am testing. Nothing about that code
strikes me yet as needing a custom DSL, nor a gem to extract. Ruby
does a great job of making this stuff easy.

Now, the above is a deliberately simple and contrived example. If we
find ourselves doing more complicated logic inside a factory method,
maybe a well-maintained and feature-rich gem such as Factory Girl can
help us there. But until I start to feel the pain of having to do
these things myself, over and over, I'm coming to realize that I'd
much rather start a new project rolling my own.

So, it seems clear to me now that the problem that Factory Girl solved
wasn't anything to do with fixtures, since it's straightforward to
create your own factory methods. It was presumably the problem of
having cumbersome factory methods that you had to write yourself.

The title of this piece is actually a bit of a lie, because factories,
or rather the things that they build, are technically fixtures,
depending on your definition of "fixture". In the language of Gerard
Meszaros, author of _xUnit Test Patterns_, the default Rails fixtures
are more specifically _shared fixtures_, meaning they are created in
the database at the start of your test suite and hang around until the
end. Factories, on the other hand, are _persistent fresh fixtures_,
meaning that they still live in the database (persistent), but their
lifecycle is confined to individual tests (fresh).

But not everyone uses this terminology, and I'm going to go with
another convention of referring to the first kind as "fixtures" from
hereon, and the second kind as "factories".

If done well, your fixtures will be a set of objects that live in the
database that together weave a kind of narrative that is revealed in
tiny installments through your unit tests. Factories, on the other
hand, allow you to build only what is needed to test your thing in
isolation, and get torn down at the end of each test.

So what was the problem with fixtures? The problem is that it obscures
important details from your test setup, putting more cognitive load on
you the programmer who has to remember all the various details of the
way the fixtures are set up. If one or more of these fixtures are the
main players in your test, you have a test smell that Meszaros calls
the "mystery guest", a specific kind of "obscure test". In his words,
"[the] test reader is not able to see the cause and effect between
fixture and verification logic because part of it is done outside the
Test Method."

Now. Back to Factory Girl. Here's a pattern that I see over and over
again in a lot of test suites:

```ruby
# app/models/user.rb
class User < ApplicationRecord
  validates :date_of_birth, presence: true

  def adult?
    (Date.today - date_of_birth) > 21.years
  end
end

# spec/factories/user.rb
FactoryGirl.define do
  factory(:user) do
    name "Alice"
    date_of_birth 30.years.ago
  end
end

# spec/models/user_spec.rb
it "tests adulthood" do
  user = create(:user)
  expect(user).to be_adult
end
```

As you can see here, the user factory defines some default value, and
the example tests some behavior that is completely dependent on the value
that's set in the factory.

(Sidebar! Now, an obvious problem with this is that if I needed to
change one of the factory default values, tests are going to break,
which should never happen. The goal of factories is to build an object
that passes validation with the minimum number of required attributes,
so you don't have to keep specifying every required attribute in every
single test you write. But if you're depending on the specific value
of any of those attributes set in the factory in your test, you're
doing it wrong.)

Here's what that would look like if we used fixtures instead:

```yaml
# spec/fixtures/users.yml
Alice:
  name: "Alice"
  date_of_birth: 30.years.ago
```

```ruby
# spec/models/user_spec.rb
it "tests adulthood" do
  user = users(:Alice)
  expect(user).to be_adult
end
```

You might observe that they look fundamentally the same. Used in this
way, factories are just a different kind of shared fixture. It will
have the same drawback of leading to obscure tests, with the bonus of
making your test suite slower because these objects have to be built
afresh for every single test. What was the point?

(Sidebar! You'll also notice that the test provides little value in
not testing around the edges (in this case dates of birth around 21
years ago). Sure, you could add some traits to address this, but
traits don't address the obscure test issue, and as they accumulate
add more and more to your cognitive load.)

Let's see if this could go any differently:

```ruby
FactoryGirl.define do
  factory(:user) do
    name "Alice"
    date_of_birth 30.years.ago
  end
end

describe "#adult?" do
  specify "a 21 year old is an adult" do
    user = create(:user, date_of_birth: 21.years.ago)
    expect(user).to be_adult
  end

  specify "a 20 year old is not an adult" do
    user = create(:user, date_of_birth: 21.years.ago - 1.day)
    expect(user).not_to be_adult
  end
end
```

Here, our factory looks exactly the same, but crucially we don't use
the default `date_of_birth` value in any of our tests that exercise
it. This means that if I changed the default value to literally
anything else that still resulted in a valid user object, my tests
would still pass. By using specific values for `date_of_birth` around
the edge of adulthood, I know that I have better tests. And by
providing those values in the test body, I can see the direct
relationship between those values and the behavior exercised.

Like a lot of sharp tools in Ruby, Factory Girl is rich with features
that are very powerful and expressive, but in my opinion prone to
overuse. In summary, don't use Factory Girl to create shared
fixtures - if that's the style you like then you may want to consider
going back to Rails fixtures instead.
