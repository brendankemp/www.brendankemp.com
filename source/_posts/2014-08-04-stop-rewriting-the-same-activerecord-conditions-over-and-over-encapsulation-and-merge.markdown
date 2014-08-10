---
layout: post
title: "Stop rewriting the same ActiveRecord conditions over and over: encapsulation and #merge"
date: 2014-08-04 13:06:02 -0400
comments: true
categories: active-record rails ruby
published: true
---

## A pain to write, a pain to read, and a pain to maintain

You've probably felt bad about writing something like this before:

```ruby
class PostsController < ApplicationController
  def index
    @posts = Post.order('created_at DESC').joins(:author).where(authors: { active: true })
  end
end
```

Queries in the controller are a pain to write, but more importantly, they are a pain to read, and a pain to maintain. Any future developer (including future you) will have to parse some raw SQL to understand which `@posts` are going to be rendered. And bizarrely, changes to `Post` or `Author` could force changes in `PostsController`—if you change `Author#active` to `Author#active_at`, or change `Post` to `has_many :authors`, or add a `Post#published_at` date… well, you get the idea.

## Keep conditions contained on the model

So how do we write scopes that are easy to read, and resilient to changes? By keeping a strict rule: conditions live in scopes on their respective models. Don't let the internals of you models leak all over your application!

This is what the controller looks like when you apply this rule:

```ruby
class PostsController < ApplicationController
  def index
    @posts = Post.with_active_authors.order_by_most_recent
  end
end
```

Now the scopes say what they do in plain language, and if any of the scope internals change, you won't have to update `PostsController`.

## Keep conditions on the correct model with #merge

Let's see what happens when we try to write the `with_active_authors` scope on the `Post` model:

```ruby
class Post < ActiveRecord::Base
  scope :with_active_authors, lambda {
    joins(:author).where(authors: { active: true })
  }
end
```

Now we have a condition for the `authors` table in the `Post` class. This is a problem for all the same reasons writing conditions in the `PostsController` was a problem. We can solve it by moving the authors condition to the `Author` class, and using `merge` to apply it in `with_active_authors`:

```ruby
class Author < ActiveRecord::Base
  scope :active, lambda {
    where(authors: { active: true })
  }
end

class Post < ActiveRecord::Base
  scope :with_active_authors, lambda {
    joins(:author).merge(Author.active)
  }
end
```

Now every condition lives on the correct model.

## You gotta keep 'em encapsulated

Many people would ask at this point: why bother? Isn't it easier to write the condition when you need it? Maybe you could extract a scope when you start reusing the same condition a second or a third time?

The problem is that it's easy to not notice when you are rewriting the same condition for the nth time. You have to remember to `grep` the project.

I also find that when a set of conditions is rewritten every time they are used, bugs occur. Something like this always happens: you remembered to order by `published_at`, but you forgot the fall-back order is `created_at` because not all records have a `published_at` date. Or any of the thousands of similar quirks that accumulate when several programmers add code on top of an imperfect data set. This is avoided if you define one canonical scope on the model itself.

Your `ActiveRecord` models are meant to encapsulate all the logic for querying their respective tables. That is their entire point. Use them! Don't be afraid to aggressively move logic into the correct class, and don't forget to use `merge` when you want to apply scopes in other classes.
