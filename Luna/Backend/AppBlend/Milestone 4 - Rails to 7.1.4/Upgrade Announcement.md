# 🎊 Official Announcement: `backend` Upgraded to Rails 7.1.4 🎊

Ladies and gents, it's my pleasure to officially inform you `backend` was successfully upgraded to Rails 7.1.4.

These are outstanding things this upgrade brought to our beloved `backend` 💗 repo.

# What does Rails 7.1 brings to the table?

Rails 7.1 was released on October 2023. This version is made of about 5000 commits and 813 people made contributions.

## Async Queries

This version of Rails brings new API in Active Record to make some queries asynchronous.

Example with `forms` table:
```ruby
promise = PatientSelfReport::Form.where(completed:true).async_count
  PatientSelfReport::Form Count (48.6ms)  SELECT COUNT(*) FROM "forms" WHERE "forms"."completed" = $1  [["completed", true]]
=> #<ActiveRecord::Promise status=complete>

promise.value
=> 22119
```

These are the async methods:

- `async_count`
- `async_sum`
- `async_minimum`
- `async_maximum`
- `async_average`
- `async_pluck`
- `async_pick`
- `async_ids`
- `async_find_by_sql`
- `async_count_by_sql`

## Set strict `locals` in views or partials

This is a way to define the only acceptable locals in partials or views. Defined with a magic comment in the top of the file.

Dummy example:
```ruby
<%# locals: (title: "Default title", comment_count: 0) %>
 
<h2><%= title %></h2>
<span class="comment-count"><%= comment_count %></span>
```

`locals:` only accepts keyword arguments. Positional or other raises an exception.

# What about Rails 8?

This bombastic version of Rails was released past November and was mainly focused on giving solo developers / small teams more tools to their Rails apps out there with simpler tooling.

The most noise things about this release are Kamal 2 + Solid Trifecta.

## Kamal 2. Modern day Capistrano

Rails 8 comes preconfigured Kamal 2.

Kamal 2 is good old Capistrano but for Docker. I haven't personally used but many people praise its simplicity to deploy RoR apps to VPS.

In DHH words:
> All it needs is the IP addresses for a set of servers with your SSH key deposited, and you’ll be ready to go into production in under two minutes.

## Solid Trifecta

Rails 8 aims to take advantage of VPS running on NVMe or SSD drives to run SQLite as the main data storage. The goal is to replace Redis with SQLite for job queues or cache.

### Solid Cable

See at -> https://github.com/rails/solid_cable

Lets us remove Redis for WebSocket functionality.

### Solid Cache

See at -> https://github.com/rails/solid_cache

Lets us get rid of Redis/Memcached for caching needs. Cache will live in disk instead of RAM. With NVMe drives speed shouldn't be affected.

### Solid Queue

See at -> https://github.com/rails/solid_queue

Lets us replaces Redis and also the job framework (Resque, Sidekiq). Works with PostgreSQL, MySQL and SQLite.

# Changes in `backend`

## Use `#to_fs` instead of `#t_s` when converting DateTime values 🗓️

Use `#to_fs` instead of old `#t_s` (without arguments) to convert Date/DateTime values to String. What does this mean?

If you use `#to_s` without an argument it would raise an ArgumentError exception. If you use it without an argument, it won't fail but won't produce the expected output.

> Note: You can also use the `DateTimeLocalizer` class when interpolating/printing date value.

## Support for exclusion constraints 🔐

We can now use Rails implementation of PostgreSQL exclusion constraints when writing migrations.

Docs -> https://guides.rubyonrails.org/v7.1/active_record_postgresql.html#exclusion-constraints

The gem that brought this feature was removed in a later PR.

## Rails.env.local? 🏠

We already have `Luna.env.local?` but this does not causes any conflict.

## Automatic inverse_of 👉🏽 👈🏽

No need to add `inverse_of` in association definitions. When the association is simple enough Rails will do this performance config by default.

Note: do not use `inverse_of` when defining an association with a custom lambda. It might not work as expected.

## Upgraded gems 🧰

These gems were upgraded to support the Rails upgrade:

- `devise_token_auth` to `1.2.3`
- `acts-as-taggable-on` to `10.0.0`
- `audited` gem to `5.4.2`
- `paranoia` gem to `2.6.3`
- `active_model_serializers` to `0.10.14`
- `active_record_extended` gem to `3.2.1`
- `bullet` to `7.1.4`
- `database_cleaner-active_record` to `2.1.0`
- `activerecord-import` to `1.5.0`
- `sentry` gems to `5.10.0`

## Links

If you want to know what more things Rails 7.1 brings to the table, check these links out:

- Release Notes -> https://guides.rubyonrails.org/v7.2.0/7_1_release_notes.html
- Posts in BigBinary blog -> https://www.bigbinary.com/blog/categories/rails-7/all
- Go Rails videos -> https://gorails.com/series/whats-new-inrails-7-1
- DHH Rails 8 post -> https://rubyonrails.org/2024/11/7/rails-8-no-paas-required