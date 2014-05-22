---
layout: post
title: "Handling Time Zones in Rails"
date: 2014-05-14 09:09
comments: true
categories: [rails, ruby, time-zones]
---

People talk about handling time zones in Rails like it's super scary and mysterious, but I don't think you should be scared. Follow a few simple principles, and Rails makes things easy:

1. Always deal with times and zones in pairs.
2. Do any time calculations in the user's time zone.
3. Interpret times from the user in the user's time zone.

## 1. Always deal with times and zones in pairs

Time values without zone info are ambiguous—avoid them. `'Jan 12, 2014, 11:00 AM'` could mean 11 AM in London or in 11 AM in Honolulu, and those events occur at very different times.

Fortunately, Rails allows us to add zones to our times with the setting `Time.zone`.

### Setting time zone in Rails

Set `Time.zone` to the user's time zone in a controller `before_filter`, and Rails will automatically convert every time into that zone. The `Time.zone` will be set for the length of the request, and every time value retrieved from the database will automatically be converted to `Time.zone`.

Say we are building a reminder app that needs to work across all time zones. The user has already entered their local time zone, and we stored it on the column `users.time_zone`:


```ruby
ApplicationController < ActionController::Base
  before_filter :set_user_time_zone

  def set_user_time_zone
    Time.zone = current_user.time_zone
    # => #<ActiveSupport::TimeZone:0x007fb80d1374d8 @name="Pacific/Honolulu" [...] >
  end
end
```

Now, for the remainder of the request, all times will be represented in HST (Hawaii Standard Time):

```ruby
reminder.remind_at
# => Mon, 14 Apr 2014 03:23:23 HST -10:00

23.hours.ago
# => Sun, 18 May 2014 04:23:37 HST -10:00
```

### Displaying times to the user

If you display to the reminder's `remind_at` time, it will be displayed correctly, in the user's time zone:

```ruby
# users/show.html.erb
Your reminder is for:
<%= reminder.remind_at.strftime('%H:%m %p, %b %d, %Y') %>
```

The time will display to the user in their local zone:

```
Your reminder is for:
03:23 AM, Apr 14, 2014
```

To get the current date and time in the local zone, use the special methods `Date.current` and `Time.current`, rather than `Time.new` and `Date.today`:

```ruby
# Server time, wrong
Time.new
# => 2014-05-19 03:23:54 -0400
Date.today
# => 2014-05-19

# User time, correct
Time.current
# => Mon, 18 May 2014 21:26:43 HST -10:00
Date.current
# => 2014-05-18
```

## 2. Calculating times in the user's time zone

Another potential trouble spot is calculating times. Suppose you want to show your user their reminders for "tomorrow". If you define tomorrow as `12:00AM-11:59PM UTC`, you are going to show the user in Hawaii the wrong information. Fortunately, once you set the time zone, Rails' convenience methods like `beginning_of_day`, `end_of_month`, etc. return the correct times:

```ruby
# beginning of the day for the server: nope
Time.now.beginning_of_day
# => 2014-05-20 00:00:00 -0400

# beginning of the day for the user: ok
Time.current.beginning_of_day
# => Tue, 20 May 2014 00:00:00 HST -10:00
```

## 3. Handling user input

Say the user from Honolulu enters the time `'1pm May 14th, 2014'`. The time they are thinking of is 1pm May 14th, HST. How do we interpret this string correctly as 1pm Hawaii Time? We can use `Time.zone.parse`:

```ruby
# Server time, wrong
Time.parse '1pm May 14th, 2014'
# => 2014-05-14 13:00:00 -0400

# User time, correct
Time.zone.parse '1pm May 14th, 2014'
# => Wed, 14 May 2014 13:00:00 HST -10:00
```

If we have a number of integers from the user (like params from a set of select fields), representing the time, we can use `Time.zone.local`:

```ruby
# Server time, wrong
Time.new(2014, 5, 14, 11, 22)
# => 2014-05-14 11:22:00 -0400

# User time, correct
Time.zone.local(2014, 5, 14, 11, 22)
# => Wed, 14 May 2014 11:22:00 HST -10:00
```

## Next time

I think these three principles are the basis for good time zone hygiene. If you keep them in mind, you should be alert to most time zone gotchas.

Next time we'll face two of the most fearsome of beasts: time zones and Postgres! Until then.
