# Fix default format for DATETIME - Notes

This needs to be fixed so that the deprecation message goes away:

```
DEPRECATION WARNING: Using a :default format for TimeWithZone#to_s is deprecated. Please use TimeWithZone#to_fs instead. If you fixed all places inside your application that you see this deprecation, you can set `ENV['RAILS_DISABLE_DEPRECATED_TO_S_CONVERSION']` to `"true"` in the `config/application.rb` file before the `Bundler.require` call to fix all the callers outside of your application.
```

# Context

One way to go about this is by removing the `:default` format altogether from the initializer. However, in Edge, by doing this form inputs are also affected and it's more difficult to handle.

We got this default in `config/initializers/date_time.rb`
```ruby
Date::DATE_FORMATS[:default] = "%m/%d/%Y"
Time::DATE_FORMATS[:default] = "%m/%d/%Y %H:%M"
```

Changing it to other than `default` will break some forms and will make dates to be printed with a different format.
```ruby
Date::DATE_FORMATS[:luxe] = "%m/%d/%Y"
Time::DATE_FORMATS[:luxe] = "%m/%d/%Y %H:%M"
```

So, what I'm doing for the AppBlend efforts is to go to every place where a date interpolation is happening and change it there. The change is to replace `to_s` with `to_fs` or add `to_fs` in string interpolations. 

There are some ways to find those places:

- Look for `to_s` calls
- Use this regex in the code editor finder: `\w+_(at|from|until)(&?\.to_date)?(\}|&?\.to_s)`

# In String Interpolations append `to_fs`

In `#{}` string interpolation, add `to_fs` to the date instance:
```ruby
[3] pry(main)> ap "hola: #{Availability.first.created_at}"
  Availability Load (0.6ms)  SELECT "availabilities".* FROM "availabilities" ORDER BY "availabilities"."id" ASC LIMIT $1  [["LIMIT", 1]]
DEPRECATION WARNING: Using a :default format for TimeWithZone#to_s is deprecated. Please use TimeWithZone#to_fs instead. If you fixed all places inside your application that you see this deprecation, you can set `ENV['RAILS_DISABLE_DEPRECATED_TO_S_CONVERSION']` to `"true"` in the `config/application.rb` file before the `Bundler.require` call to fix all the callers outside of your application. (called from <main> at bin/rails:4)
"hola: 09/06/2018 16:04"
=> nil
[4] pry(main)> ap "hola: #{Availability.first.created_at.to_fs}"
  Availability Load (0.8ms)  SELECT "availabilities".* FROM "availabilities" ORDER BY "availabilities"."id" ASC LIMIT $1  [["LIMIT", 1]]
"hola: 09/06/2018 16:04"
```

# Replace `to_date` with DateTimeLocalizer

Replacing the `:default` to `:luxe` makes some test break:
```bash
Failure/Error:
       expect(data[:node][:carePlans]).to eq(
         [
           name: "#{episode.injury.name} (#{appt1.scheduled_date.to_date} - #{appt2.scheduled_date.to_date})"
         ]
       )

       expected: [{:name=>"injury159664 (2018-10-07 - 2018-10-21)"}]
            got: [{:name=>"injury159664 (10/07/2018 - 10/21/2018)"}]
```

When using `to_fs(:luxe)` instead of `to_date` still does not work:
```bash
Failure/Error:
       expect(data[:node][:carePlans]).to eq(
         [
           name: "#{episode.injury.name} (#{appt1.scheduled_date.to_fs(:luxe)} - #{appt2.scheduled_date.to_fs(:luxe)})"
         ]
       )

       expected: [{:name=>"injury161094 (10/07/2018 11:00 - 10/21/2018 11:00)"}]
            got: [{:name=>"injury161094 (10/07/2018 - 10/21/2018)"}]
```

It's using the format in Time constant.

## Is this code using the default format?

Without the `to_date` the printed date is:
```ruby
Sun, 07 Oct 2018 11:00:00.000000000 UTC +00:00
```

With the `to_date`:
```ruby
Sun, 07 Oct 2018
```

## What is `to_date`?

[Docs](https://apidock.com/rails/String/to_date).

> Converts a string to a Date value.

```ruby
"1-1-2012".to_date   # => Sun, 01 Jan 2012
"01/01/2012".to_date # => Sun, 01 Jan 2012
"2012-12-13".to_date # => Thu, 13 Dec 2012

"12/13/2012".to_date # => ArgumentError: invalid date
```

## How to fix?

Use the `DateTimeLocalizer` class with `american` method. For example:
```ruby
DateTimeLocalizer.new(appt1.scheduled_date, appt1.region).american
#=> 10/07/2018
```

# When using `to_date` append `to_fs`

Another way to go around places using `to_date` is to append `to_fs`:
```ruby
[2] pry(main)> "#{Workout.first.created_at.to_date}"
  Workout Load (0.9ms)  SELECT "workouts".* FROM "workouts" ORDER BY "workouts"."id" ASC LIMIT $1  [["LIMIT", 1]]
DEPRECATION WARNING: Using a :default format for Date#to_s is deprecated. Please use Date#to_fs instead. If you fixed all places inside your application that you see this deprecation, you can set `ENV['RAILS_DISABLE_DEPRECATED_TO_S_CONVERSION']` to `"true"` in the `config/application.rb` file before the `Bundler.require` call to fix all the callers outside of your application. (called from <main> at bin/rails:4)
=> "12/15/2023"
[3] pry(main)> "#{Workout.first.created_at.to_date.to_fs}"
  Workout Load (0.9ms)  SELECT "workouts".* FROM "workouts" ORDER BY "workouts"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> "12/15/2023"
```