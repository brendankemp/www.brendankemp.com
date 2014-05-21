---
layout: post
title: "Handling Timezones in Postgres and Rails"
date: 2014-05-14 09:09
comments: true
categories: [rails, ruby, postgres]
---

I want to show 3 basic principles of dealing with time zones:

1. interpret times from the user in the user's time zone
2. always deal with times and zones in pairs.
3. do a bunch of wacky things to play nice with Postgres

The last one isn't a principle so much as something you have to watch out for when you think about timezones.

### Setting time zone in Rails

Set `Time.zone` to the user's timezone in a controller before filter, and Rails will automatically convert every time into that zone. The `Time.zone` will last for the length of the request, and every time retrieved from the database will automatically be represented in that zone.

Say we have asked the user for their timezone, and stored it on the column `users.timezone`:


```ruby
ApplicationController < ActionController::Base
  before_filter :set_user_timezone

  def set_user_timezone
    Time.zone = current_user.timezone # => #<ActiveSupport::TimeZone:0x007fb80d1374d8 @name="Pacific/Honolulu"
  end
end
```

For the remainder of the request, all times will be represented in 'HST':

```ruby
user.created_at # => Mon, 14 Apr 2014 03:23:23 HST -10:00
23.hours.ago # => Sun, 18 May 2014 04:23:37 HST -10:00
```

#### Displaying times to the user

If you display to the user their created_at time, it will be displayed correctly, in their time zone:

```ruby
# users/show.html.erb
You signed up at: <%= current_user.created_at.strftime('%H:%m %p, %b %d, %Y') %>
```

That will display to the user in their local zone:

```
You signed up at: 03:23 AM, Apr 14, 2014
```

You can get the current date and time in the local zone too, but you need to use the special methods `Date.current` and `Time.current`, rather than `Time.new`

```ruby
# Server time, wrong
Time.new # => 2014-05-19 03:23:54 -0400
Date.today # => 2014-05-19

# User time, correct
Time.current # => Mon, 18 May 2014 21:26:43 HST -10:00
Date.current # => 2014-05-18
```

Now you have time represented in the user's local time. That's good. You can display local times to the user.

#### Calculating times in the user's time zone

You can also use Rails convenience methods like `beginning_of_day`, `end_of_month`, etc, and get the get the correct time:

```ruby
# beginning of the day for the server
Time.now.beginning_of_day # => 2014-05-20 00:00:00 -0400
# beginning of the day for the user
Time.current.beginning_of_day # => Tue, 20 May 2014 00:00:00 HST -10:00
```

If you do any day-based calculations, (like counting the user's tweets so far today), it's usually better to define the boundaries of the day as the user understands them.

What about handling user input?

### Handling user input

Say the user from Honolulu enters the time '1pm May 14th, 2014'. The time they are thinking of is 1pm May 14th, Hawaii Standard Time. How do we interpret this string correctly as 1pm Hawaii Time? We can use `Time.zone.parse`:

```ruby
# Server time, wrong
Time.parse '1pm May 14th, 2014' # => 2014-05-14 13:00:00 -0400
# User time, correct
Time.zone.parse '1pm May 14th, 2014' # => Wed, 14 May 2014 13:00:00 HST -10:00
```

If we have a number of integers from the user (like params from a set of select fields), representing the time, we can use `Time.zone.local`:

```ruby
# Server time, wrong
Time.new(2014, 5, 14, 11, 22) # => 2014-05-14 11:22:00 -0400
# User time, correct
Time.zone.local(2014, 5, 14, 11, 22) # => Wed, 14 May 2014 11:22:00 HST -10:00
```

### Communicating times to Postgres

We know how to interpret, display, and calculate times in a user's time. What about saving those times to the database, or querying times in the database? It actually works mostly how you would like.

We already saw that Rails will change times to the `Time.zone` when retrieving them from the database. What about writing them to the database? Take this example from a reminder app:

```ruby
reminder = Reminder.first
reminder.remind_at = Time.zone.parse('2014-06-21 12:30pm') # => Sat, 21 Jun 2014 12:30:00 HST -10:00
reminder.save # => UPDATE "reminders" SET "remind_at" = '2014-06-21 22:30:00.000000' …
reminder.reload.remind_at # => Sat, 21 Jun 2014 12:30:00 HST -10:00
```

Notice that when the time is written to the database, it is stored in UTC, but when we reload the reminder and return `remind_at`, Rails converts to HST again. `update_column` also works:

```ruby
reminder.update_column :remind_at, Time.zone.parse('2014-08-21 10:30am')
# => UPDATE "reminders" SET "remind_at" = '2014-08-21 20:30:00.000000'
reminder.reload.remind_at # => Thu, 21 Aug 2014 10:30:00 HST -10:00
```

### Querying times in Postgres

When possible, use ActiveRecord's safe interpolation to build time queries. It will correctly handle the time zones for you.

Let's try to retrieve that reminder that we just updated:

#### Safe Interpolation

Safe interpolation works:
```ruby
Reminder.where("remind_at < ?", Time.zone.parse('2014-08-21 12:00pm'))
# Reminder Load (2.4ms)  SELECT "reminders".* FROM "reminders" WHERE (remind_at < '2014-08-21 22:00:00.000000')
# => [#<Reminder id: …>]
```

We query for reminders before 12 noon on August 21. Our reminder is for 10:30am on that day, so it is returned, as we expect.

Remember that times are stored in UTC in the database, so Rails converts your time from HST to UTC when building the SQL query.

#### Standard Ruby Interpolation
What happens if we try to interpolate with standard Ruby string interpolation?

```ruby
Reminder.where("remind_at < '#{Time.zone.parse('2014-08-21 12:00pm')}'")
#  Reminder Load (0.5ms)  SELECT "reminders".* FROM "reminders" WHERE (remind_at < '2014-08-21 12:00:00 -1000')
# => []
```

Standard interpolation does not convert the time to UTC, and Postgres doesn't know to interpret it as HST. Postgres runs a query for reminders before 12 Noon UTC, which is 10 hours before 12 Noon HST, the time that we meant. You should be using safe interpolation anyway, so it's nice to know that it works. If for some reason you need to use regular string interpolation, you can tell Postgres to interpret the string as a time and zone with the Postgres type `TIMESTAMP WITH TIME ZONE`.

```ruby
Reminder.where("remind_at < TIMESTAMP WITH TIME ZONE '#{Time.zone.parse('2014-08-21 12:00pm')}'")
#  Reminder Load (2.1ms)  SELECT "reminders".* FROM "reminders" WHERE (remind_at < TIMESTAMP WITH TIME ZONE '2014-08-21 12:00:00 -1000')
=> [#<Reminder id: …>]
```

Postgres correctly interprets `'2014-08-21 12:00:00 -1000'` as 12 Noon HST (or 10pm UTC), and returns the correct reminder.

#### Using Postgres' DATE_TRUNC function with Rails

tl;dr: You have to convert times with `"remind_at::TIMESTAMPTZ AT TIME ZONE '#{Time.zone.formatted_offset}'::INTERVAL"` before passing them to the DATE_TRUNC function, if you want the correct dates for the user's timezone.

Say you want to do a calender view for your reminder app. You want to return a list of days, and a list of the names of the reminders for each day. You want a speedy query, so you will group by day in SQL using Postgres' `DATE_TRUNC` function. How will you group by days in the user's timezone, rather than the database's timezone?

Say you have a user on Hawaii time, and they have a reminder for 5pm HST, August 21st. You try to your DATE_TRUNC query for reminders that week, and this is what you get:

```
time = Time.zone.parse('2014-08-21 17:00:00') # => Thu, 21 Aug 2014 17:00:00 HST -10:00
Reminder.create(:remind_at => time, :name => 'Go Running!')
reminders = Reminder.select("
    ARRAY_AGG(name) AS reminder_names,
    DATE_TRUNC('Day', remind_at) AS reminder_day
  ").
  group('reminder_day')
reminders.map{|r| [r.reminder_day, r.reminder_names] }
# => [["2014-08-22 00:00:00", "{\"Go Running!\"}"]]
```

The reminder is showing up for the 22nd, not the 21st. What happened? In Postgres, the time is stored in 2014-08-22 03:00:00 UTC. When `DATE_TRUNC('Day')` truncates the time values, what we get is '2014-08-22 00:00:00'. We need to convert the time to HST **before** we truncate the time values. For this purpose we can use Postgres' `TIMESTAMPTZ AT TIME ZONE`

```ruby
time = Time.zone.parse('2014-08-21 17:00:00') # => Thu, 21 Aug 2014 17:00:00 HST -10:00
Reminder.create(:remind_at => time, :name => 'Go Running!')
reminders = Reminder.select("
    ARRAY_AGG(name) AS reminder_names,
    DATE_TRUNC(
      'Day',
      remind_at::TIMESTAMPTZ AT TIME ZONE '#{Time.zone.formatted_offset}'::INTERVAL
    ) AS reminder_day
  ").
  group('reminder_day')
reminders.map{|r| [r.reminder_day, r.reminder_names] }
# => [["2014-08-21 00:00:00", "{\"Go Running!\"}"]]
```

Now the reminder is showing up on the 21st, which is correct for our user in Hawaii. What's going on?

#### Digression: TIMESTAMP WITHOUT TIME ZONE

Rails actually uses TIMESTAMP WITHOUT TIME ZONE columns in Postgres to store datetimes.

```sql
> rails dbconsole
psql (9.3.2)
Type "help" for help.

reminders_dev=# \d+ reminders
                                                         Table "public.reminders"
   Column   |            Type             |                       Modifiers                        | Storage  | Stats target | Description
------------+-----------------------------+--------------------------------------------------------+----------+--------------+-------------
 id         | integer                     | not null default nextval('reminders_id_seq'::regclass) | plain    |              |
 name       | character varying(255)      |                                                        | extended |              |
 remind_at  | timestamp without time zone |                                                        | plain    |              |
 created_at | timestamp without time zone | not null                                               | plain    |              |
 updated_at | timestamp without time zone | not null                                               | plain    |              |
Indexes:
    "reminders_pkey" PRIMARY KEY, btree (id)
Has OIDs: no
```

Normally, this would not make sense, because a timestamp without a timezone does not make sense. '2014-05-29 12:00:00' (without time zone info) could be one of dozens of different times in all the time zones across the globe. There's no way to know what time is meant, without the time zone. Rails gets around this by storing everything in UTC. The database column doesn't include time zone information, but the time zone is implicitly UTC.

#### Mashing time zone info into the db

In order to do our DATE_TRUNC calculation, we have to convert the time-zone-less (implicitly UTC) time values to the local time first. We can do this with the Postgres typecast TIMESTAMPTZ AT TIME ZONE. TIMESTAMPTZ is a timestamp with a time zone included. TIMESTAMPTZ AT TIME ZONE requires a time zone:

```sql
// remind_at in UTC
# SELECT remind_at FROM reminders LIMIT 1;
      remind_at
---------------------
 2014-08-22 03:00:00
(1 row)
// remind_at converted to HST
# SELECT remind_at::TIMESTAMPTZ AT TIME ZONE INTERVAL '-10:00'::INTERVAL FROM reminders LIMIT 1;
      timezone
---------------------
 2014-08-21 21:00:00
(1 row)
```

Fortunately, you can pass the time zone as a Postgres INTERVAL, which can easily be created in from a string: `'-10:00'::INTERVAL`. Even more luckily, Rails offers `Time.zone.formatted_offset` to return the current user's time zone in exactly that format.

```ruby
Time.zone.formatted_offset
# => "-10:00"
```

We interpolate the formatted_offset into the SQL query for the solution:

```ruby
reminders = Reminder.select("remind_at::TIMESTAMPTZ AT TIME ZONE INTERVAL '#{Time.zone.formatted_offset}'::INTERVAL AS remind_at_in_time_zone")
reminders.map &:remind_at_in_time_zone
#=> ["2014-08-21 17:00:00"]
```

Yay! That's the correct reminder time for our user in Hawaii. But wait—that's a string. The last step is to parse the string as a Ruby `Time`, and we will have the correct form of information we need.

```ruby
reminders.map{|r| Time.zone.parse(r.remind_at_in_time_zone) }
#=> [Thu, 21 Aug 2014 17:00:00 HST -10:00]
```

We break out `Time.zone.parse`, and Ruby parses it correctly, as a Hawaii time. Does this work with our `DATE_TRUNC` example problem?

```ruby
days_with_reminders = Reminder.select("
    ARRAY_AGG(name) AS reminder_names,
    DATE_TRUNC(
      'Day',
      remind_at::TIMESTAMPTZ AT TIME ZONE '#{Time.zone.formatted_offset}'::INTERVAL
    ) AS reminder_day
  ").
  group('reminder_day')
days_with_reminders.map{|r| [Time.zone.parse(r.reminder_day), r.reminder_names] }
#=> [[Thu, 21 Aug 2014 00:00:00 HST -10:00, "{\"Go Running!\"}"]]

```

Yup! We get the 21st in HST, which is exactly what we want. We could call `to_date` and get `Thu, 21 Aug 2014` to indicate that we mean the whole day, and not any specific time. We can now display reminders for the correct day.
