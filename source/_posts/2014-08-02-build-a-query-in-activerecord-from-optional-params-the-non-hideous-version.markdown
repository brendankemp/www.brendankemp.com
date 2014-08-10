---
layout: post
title: "Build a query in ActiveRecord from optional params: the non-hideous version"
date: 2014-08-02 23:55:18 -0400
comments: true
categories: active-record rails ruby
published: true
---

So you are building a search form. It produces a hash of query params, all of which are optional.
How do you build up the query in ActiveRecord? Do you nil-check every param?

```ruby
class Person < ActiveRecord::Base
  def self.search(name: nil, email: nil, start_date: nil, end_date: nil)
    query = scoped

    if name.present?
      query = query.where(name: name)
    end

    if email.present?
      query = query.where(email: email)
    end

    if start_date.present?
      query = query.where('created_at > ?', start_date)
    end

    if end_date.present?
      query = query.where('created_at < ?' end_date)
    end

    query
  end
end
```

Ick. This should feel bad to you.
Every param repeats the pattern of [check if param is present], if so [apply param to query].
That adds tons of visual noise. We can make it way more expressive.

```ruby
class Person < ActiveRecord::Base
  def self.search(name: nil, email: nil, start_date: nil, end_date: nil)
    with_name(name)
      .with_email(email)
      .signed_up_after(start_date)
      .signed_up_before(end_date)
  end

  scope :with_name, proc { |name|
    if name.present?
      where(name: name)
    end
  }
  scope :with_email # …

  scope :signed_up_after, proc { |date|
    if date.present?
      where('created_at > ?', date)
    end
  }
  scope :signed_up_before # …
end
```

That's the entire solution. Read on if you want to learn more about it.

### Why is the second solution any better?

Let's compare the application of one parameter in the `search` method:

From… 

```ruby
if start_date.present?
  query = query.where('created_at > ?', start_date)
end
```

To…

```ruby
.signed_up_after(start_date)
```

That's massively more expressive! We cut out all the cruft of nil-checking, and query-building.
We're left with a single, expressive method call. But the cruft hasn't disappeared. We've moved it:

```ruby
scope :signed_up_after, lambda { |date|
  if date.present?
    where('created_at > ?', date)
  end
}
```

The query-building noise (`query = query.where()`) is gone completely.
We are still left with nil-checking (`if date.present?; end`).
How is that any better?
Well, now it is encapsulated in its own method that is only 3 lines, and thus can be understood at a glance.
The method can be reused without re-writing the nil-checking logic.

### ActiveRecord saves us, once again

But wait. You may have noticed something odd about the `signed_up_after` scope.
If `date` is nil, then `if date.present?` will return nil, and the lambda will return nil.
If we try to chain other calls on after it, then it will blow up!

```ruby
Person.signed_up_after(nil) # => nil???
Person.signed_up_after(nil).name('Barry') # => Error: method #name called on nil ????
```

But ActiveRecord takes care of this for us.
If the scope lambda returns nil, then it will return the scoped object, so other scope calls can be chained after.
It's equivalent to this class method:

```ruby
def self.signed_up_after(date)
  return scoped if date.blank?

  where('created_at > ?', date)
end
```

If you use the `scope` method, then that's one less then you have to repeat for every query method.

### Why use lambda for the scope?

Because it forces us to use the correct number of params.

```ruby
class Person < ActiveRecord::Base
  scope :with_name, lambda { |name| where(name: name) }
end
Person.with_name('Fred') # => fine
Person.with_name         # => BOOM
```

Proc params are optional:

```ruby
class Person < ActiveRecord::Base
  scope :with_name, proc { |name| where(name: name) }
end
Person.with_name('Fred') # => fine
Person.with_name         # => also fine
```

If I call `with_name`, and forget to pass a param, there's 99% chance that's a mistake.
I'd like hear about that in the form of an exception!
