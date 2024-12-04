# Fix default format for DATETIME

We got this default in `config/initializers/date_time.rb`

```ruby
Date::DATE_FORMATS[:default] = "%m/%d/%Y"
Time::DATE_FORMATS[:default] = "%m/%d/%Y %H:%M"
```

Changing it to other than `default`

```ruby
Date::DATE_FORMATS[:luxe] = "%m/%d/%Y"
Time::DATE_FORMATS[:luxe] = "%m/%d/%Y %H:%M"
```

makes some test break:
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

When using `to_fs(:luxe)` instead of `to_date`
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
