# Upgrade to Rails 7.1.4 Announcement

Ladies and gents, it's my pleasure to officially inform you `backend` was successfully upgrade to Rails 7.1.4.

Following is a list of outstanding things this upgrade brings to our beloved repo.

## Use `to_fs` instead of `t_s` when converting DateTime values

Use `to_fs` instead of old `t_s` (without arguments) to convert DateTime values to String.

## Support for exclusion constraints

We can now use Rails implementation of exclusion constraints when writing migrations.

## Rails.env.local?

We already have Luna.env.local?

## Automatic inverse_of

No need to add inverse_of in association definitions. When the association is simple enough Rails will do this performance config by default.

Note: do not use inverse_of when defining an association with a custom lambda.