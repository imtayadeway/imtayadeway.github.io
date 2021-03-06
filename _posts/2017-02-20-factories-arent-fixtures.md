---
layout: post
title: Factories aren't Fixtures
tags:
- testing
- ruby
date: 2017-02-20 11:48:07
---

As someone who learned both to program and to test for the first time
with Rails, I was quickly exposed to a lot of opinions about testing
at once, with a lot of hand-waving. One of these was, as I remember
it, that Rails tests with fixtures by default, that fixtures are
problematic, that Factory Girl is a solution to those problems, so we
just use Factory Girl. I probably internalized this at the time as
"use Factory Girl to build objects in tests" without really
questioning why.

Some years later now, I sincerely regret _not_ learning to use
fixtures first, to experience those pains for myself (or not), to find
out to what problem exactly Factory Girl was a solution. For, I've
come to discover, Factory Girl doesn't prevent you from having some of
the same issues that you'd find with fixtures.

To understand this a bit better, let's do a simple refactoring from
fixtures to factories to demonstrate what problems we are solving
along the way.

Consider the following:

```ruby
# app/models/user.rb
class User < ApplicationRecord
  validates :name, presence: true
  validates :date_of_birth, presence: true

  def adult?
    date_of_birth + 21.years >= Date.today
  end
end
```

```yaml
# spec/fixtures/users.yml
Alice:
  name: "Alice"
  date_of_birth: <%= 21.years.ago %>
Bob:
  name: "Bob"
  date_of_birth: <%= 21.years.ago - 1.day %>
```

```ruby
# spec/models/user_spec.rb
specify "a person of > 21 years is an adult" do
  user = users(:Alice)
  expect(user).to be_adult
end

specify "a person of < 21 years is not an adult" do
  user = users(:Bob)
  expect(user).not_to be_adult
end
```

Here we have two fixtures that contrast two different kinds of
user. If done well, your fixtures will be a set of objects that live
in the database that together weave a kind of narrative that is
revealed in tiny installments through your unit tests. Elsewhere in
our test suite, we'd continue with this knowledge that Alice is an
adult and Bob is a minor.

So what's the problem? Well, one is what Meszaros calls the "mystery
guest", a kind of "obscure test" smell. What that means is that the
main players in our tests - Alice and Bob, are defined far off in the
`spec/fixtures/users.yml` file. Just looking at the test body, it's
hard to know exactly what it was about Alice and Bob that made one an
adult, the other not. (Sure, we should know the rules about adulthood
in whatever country we're in, but it's easy to see how a slightly more
complicated example might not be so clear).

Let's try to address that concern head on by removing the fixtures:

```ruby
# spec/models/user_spec.rb
specify "a person of > 21 years is an adult" do
  user = User.create!(name: "Alice", date_of_birth: 21.years.ago)
  expect(user).to be_adult
end

specify "a person of < 21 years is not an adult" do
  user = User.create!(name: "Bob", date_of_birth: 21.years.ago - 1.day)
  expect(user).not_to be_adult
end
```

We've solved the mystery guest problem! Now we can see at a glance
what the relationship is between the attributes of each user and the
behavior exhibited by them.

Unfortunately, we have a new problem. Because a user requires a
`:name` attribute, we have to specify a name in order to build a valid
user object in each test (we might in certain instances be able to get
away with using invalid objects, but it is probably not a good
idea). Here, the fact that we've had to give our users names has given
us another obscure test smell - we have introduced some noise in that
it's not clear at a glance _which_ attributes were relevant to the
behavior that's getting exercised.

Another problem that might emerge is if we added a new attribute to
`User` that was validated against - every test that builds a user
could fail for reasons that could be wholly unrelated to the behavior
they are trying to exercise.

Let's try this again, extracting out a factory method:

```ruby
# spec/models/user_spec.rb
specify "a person of > 21 years is an adult" do
  user = create_user(date_of_birth: 21.years.ago)
  expect(user).to be_adult
end

specify "a person of < 21 years is not an adult" do
  user = create_user(date_of_birth: 21.years.ago - 1.day)
  expect(user).not_to be_adult
end

def create_user(attributes = {})
  User.create!({name: "Alice", date_of_birth: 30.years.ago}.merge(attributes))
end
```

Problem solved! We have some sensible defaults in the factory method,
meaning that we don't have to specify attributes that are not relevant
in every test, and we've overridden the one that we're testing -
`date_of_birth` - in those tests on adulthood. If new validations are
added, we have one place to update to make our tests pass again.

I'm going to pause here for some reflection before we complete our
refactoring. There is another thing that I regret about the way I
learned to test. And it is simply not using my own factory methods as
I have above, before finding out what problem Factory Girl was trying
to address with doing _that_. Nothing about the code above strikes me
yet as needing a custom DSL, or a gem to extract. Ruby already does a
great job of making this stuff easy.

Sure, the above is a deliberately simple and contrived example. If we
find ourselves doing more complicated logic inside a factory method,
maybe a well-maintained and feature-rich gem such as Factory Girl can
help us there. Let's assume that we've reached that point and plough
on so we can complete the refactoring.

```ruby
# spec/factories/user.rb
FactoryGirl.define do
  factory :user do
    name "Alice"
    date_of_birth 30.years.ago
  end
end
```

```ruby
# spec/models/user_spec.rb
specify "a person of > 21 years is an adult" do
  user = create(:user, date_of_birth: 21.years.ago)
  expect(user).to be_adult
end

specify "a person of < 21 years is not an adult" do
  user = create(:user, date_of_birth: 21.years.ago - 1.day)
  expect(user).not_to be_adult
end
```

This is fine. Our tests look pretty much the same as before, but
instead of a factory method we have a Factory Girl factory. We haven't
solved any immediate problems in this last step, but if our `User`
model gets more complicated to set up, Factory Girl will be there with
lots more features for handling just about anything we might want to
throw at it.

It seems clear to me now that the problem that Factory Girl solved
wasn't anything to do with fixtures, since it's straightforward to
create your own factory methods. It was presumably the problem of
having cumbersome factory methods that you had to write yourself.

However. This is not quite the end of the story for some folks, and
that there's a further refactoring we can seize upon:

```ruby
# spec/factories/user.rb
FactoryGirl.define do
  factory :user do
    name "Alice"
    date_of_birth 30.years.ago

    trait :adult do
      date_of_birth 21.years.ago
    end

    trait :minor do
      date_of_birth 21.years.ago - 1.day
    end
  end
end
```

```ruby
# spec/models/user_spec.rb
specify "a person of > 21 years is an adult" do
  user = create(:user, :adult)
  expect(user).to be_adult
end

specify "a person of < 21 years is not an adult" do
  user = create(:user, :minor)
  expect(user).not_to be_adult
end
```

Here, we've used Factory Girl's traits API to define what it means to
be both an adult and a minor in the factory itself, so if we ever have
to use that concept again the knowledge for how to do that is
contained in one place. Well done to us!

But hang on. Haven't we just reintroduced the mystery guest smell that
we were trying so hard to get away from? You might observe that these
tests look fundamentally the same as the ones that we started out
with.

Used in this way, factories are just a different kind of shared
fixture. We have the same drawback of having test obscurity, and we've
taken the penalty of slower tests because these objects have to be
built afresh for every single example. What was the point?

Okay, okay. Traits are more of an advanced feature in Factory
Girl. They might be useful, but they don't solve any problems that we
have at this point. How about we just keep things simple:

```ruby
# spec/factories/user.rb
FactoryGirl.define do
  factory :user do
    name "Alice"
    date_of_birth 30.years.ago
  end
end
```

```ruby
# spec/models/user_spec.rb
it "tests adulthood" do
  user = create(:user)
  expect(user).to be_adult
end
```

This example is actually worse, and is quite a popular
anti-pattern. An obvious problem is that if I needed to change one of
the factory default values, tests are going to break, which should
never happen. The goal of factories is to build an object that passes
validation with the minimum number of required attributes, so you
don't have to keep specifying every required attribute in every single
test you write. But if you're depending on the specific value of any
of those attributes set in the factory in your test, you're Doing It
Wrong ™️.

You'll also notice that the test provides little value in not testing
around the edges (in this case dates of birth around 21 years
ago).

Let's compare with our earlier example (the one before things started
to go wrong):

```ruby
# spec/factories/user.rb
FactoryGirl.define do
  factory :user do
    name "Alice"
    date_of_birth 30.years.ago
  end
end
```

```ruby
# spec/models/user_spec.rb
specify "a person of > 21 years is an adult" do
  user = create(:user, date_of_birth: 21.years.ago)
  expect(user).to be_adult
end

specify "a person of < 21 years is not an adult" do
  user = create(:user, date_of_birth: 21.years.ago - 1.day)
  expect(user).not_to be_adult
end
```

Crucially we don't use the default `date_of_birth` value in any of our
tests that exercise it. This means that if I changed the default value
to literally anything else that still resulted in a valid user object,
my tests would still pass. By using specific values for
`date_of_birth` around the edge of adulthood, I know that I have
better tests. And by providing those values in the test body, I can
see the direct relationship between those values and the behavior
exercised.

Like a lot of sharp tools in Ruby, Factory Girl is rich with features
that are very powerful and expressive. But in my opinion, its more
advanced features are prone to overuse. It's also easy to confuse
Factory Girl for a library for creating shared fixtures - Rails
already comes with one, and it's better at doing that. Neither of
these are faults of Factory Girl, rather I believe they are faults in
the way we teach testing.

So don't use Factory Girl to create shared fixtures - if that's the
style you like then you may want to consider going back to Rails'
fixtures instead.
