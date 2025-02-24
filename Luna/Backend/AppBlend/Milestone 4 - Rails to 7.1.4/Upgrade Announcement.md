# 🎊 Official Announcement: `backend` Upgraded to Rails 7.1.4 🎊

Ladies and gents, it's my pleasure to officially inform you `backend` was successfully upgraded to Rails 7.1.4.

These are outstanding things this upgrade brought to our beloved `backend` 💗 repo.

# What does Rails 7.1 brings to the table?

cosas sobre esta version

# What about Rails 8?

cosas sobre esta version

# Changes in `backend`

## Use `#to_fs` instead of `#t_s` when converting DateTime values 🗓️

Use `#to_fs` instead of old `#t_s` (without arguments) to convert Date/DateTime values to String. What does this mean?

If you use `#to_s` without an argument it would raise an ArgumentError exception. If you use it without an argument, it won't fail but won't produce the expected output.

> Note: You can also use the `DateTimeLocalizer` class when interpolating/printing date value.

## Support for exclusion constraints 🔐

We can now use Rails implementation of PostgreSQL exclusion constraints when writing migrations.

Docs -> https://guides.rubyonrails.org/v7.1/active_record_postgresql.html#exclusion-constraints

The gem that brought this feature was removed in a posterior PR.

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

- https://www.bigbinary.com/blog/categories/rails-7/all
- https://gorails.com/series/whats-new-inrails-7-1