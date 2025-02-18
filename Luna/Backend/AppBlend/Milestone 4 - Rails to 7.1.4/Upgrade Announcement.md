# 🎊 Upgrade to Rails 7.1.4 Announcement 🎊

Ladies and gents, it's my pleasure to officially inform you `backend` was successfully upgrade to Rails 7.1.4.

Following is a list of outstanding things this upgrade brings to our beloved repo.

## Use `to_fs` instead of `t_s` when converting DateTime values 🗓️

Use `to_fs` instead of old `t_s` (without arguments) to convert DateTime values to String.

## Support for exclusion constraints 🔐

We can now use Rails implementation of PostgreSQL exclusion constraints when writing migrations.

Docs -> https://guides.rubyonrails.org/v7.1/active_record_postgresql.html#exclusion-constraints

## Rails.env.local? 🏠

We already have `Luna.env.local?` but this does not causes any conflict.

## Automatic inverse_of 👉🏽 👈🏽

No need to add `inverse_of` in association definitions. When the association is simple enough Rails will do this performance config by default.

Note: do not use `inverse_of` when defining an association with a custom lambda.

## Upgraded gems 🧰

This is the list of upgraded gems:

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


