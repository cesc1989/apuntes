# Upgrade Edge to Rails 7.2

Ryan wants this upgrade to fix performance issues.

> [!Info]
> Upgrading From 7.0 to 7.1 https://guides.rubyonrails.org/v7.1/upgrading_ruby_on_rails.html#upgrading-from-rails-7-0-to-rails-7-1

## ✅ Start with Rails 7.0.8.4 ✅

~~Let's start with this version to get closest to 7.1.0.~~ Done

## Rails 7.1.4 to 7.2.1

Make a list of items present in the changelog to put an eye on and discuss with Edge team.

### Rails 7.1 Releases

Releases:
- 7.1.0 (Oct 5, 2023) https://github.com/rails/rails/releases/tag/v7.1.0
- 7.1.1 (Oct 11, 2023) https://github.com/rails/rails/releases/tag/v7.1.1
- 7.1.2 (Nov 10, 2023) https://github.com/rails/rails/releases/tag/v7.1.2
- 7.1.3 (Jan 16, 2024) https://github.com/rails/rails/releases/tag/v7.1.3
- 7.1.4 (Aug 22, 2024) https://github.com/rails/rails/releases/tag/v7.1.4

### Rails 7.2 Release

Releases:
- 7.2.0 (Aug 9, 2024) https://github.com/rails/rails/releases/tag/v7.2.0
- 7.2.1 (Aug 22, 2024) https://github.com/rails/rails/releases/tag/v7.2.1

# Changes in 7.1.0 through 7.1.4

## 7.1.0

**Active Support**

Remove deprecated support to generate incorrect RFC 4122 UUIDs when providing a namespace ID that is not one of the  
constants defined on `Digest::UUID`.

Deprecate `config.active_support.use_rfc4122_namespaced_uuids`.

---

Deprecate `config.active_support.disable_to_s_conversion`.

Remove deprecated option to passing a format to `#to_s` in `Array`, `Range`, `Date`, `DateTime`, `Time`,  
`BigDecimal`, `Float` and, `Integer`.

---

`config.i18n.raise_on_missing_translations = true` now raises on any missing translation.

Previously it would only raise when called in a view or controller. Now it will raise  
anytime `I18n.t` is provided an unrecognised key.

If you do not want this behaviour, you can customise the i18n exception handler. See the  
upgrading guide or i18n guide for more information.

---

Add `Rails.env.local?` shorthand for `Rails.env.development? || Rails.env.test?`.

---

`ActiveSupport::Cache:Store#fetch` now passes an options accessor to the block.

It makes possible to override cache options:

```ruby
Rails.cache.fetch("3rd-party-token") do |name, options|
  token = fetch_token_from_remote
  # set cache's TTL to match token's TTL
  options.expires_in = token.expires_in
  token
end
```

---

Deprecate preserving the pre-Ruby 2.4 behavior of `to_time`

With Ruby 2.4+ the default for +to_time+ changed from converting to the  
local system time to preserving the offset of the receiver. At the time Rails  
supported older versions of Ruby so a compatibility layer was added to assist  
in the migration process. From Rails 5.0 new applications have defaulted to  
the Ruby 2.4+ behavior and since Rails 7.0 now only supports Ruby 2.7+  
this compatibility layer can be safely removed.

To minimize any noise generated the deprecation warning only appears when the  
setting is configured to `false` as that is the only scenario where the  
removal of the compatibility layer has any effect.

---

**Active Record**

Better naming for unique constraints support.

Naming unique keys leads to misunderstanding it's a short-hand of unique indexes.  
Just naming it unique constraints is not misleading.

In Rails 7.1.0.beta1 or before:

```ruby
add_unique_key :sections, [:position], deferrable: :deferred, name: "unique_section_position"
remove_unique_key :sections, name: "unique_section_position"
```

Now:

```ruby
add_unique_constraint :sections, [:position], deferrable: :deferred, name: "unique_section_position"
remove_unique_constraint :sections, name: "unique_section_position"
```

---

Fix unscope is not working in specific case

Before:

```ruby
Post.where(id: 1...3).unscope(where: :id).to_sql # "SELECT `posts`.* FROM `posts` WHERE `posts`.`id` >= 1 AND `posts`.`id` < 3"
```

After:

```ruby
Post.where(id: 1...3).unscope(where: :id).to_sql # "SELECT `posts`.* FROM `posts`"
```

---

Introduce a more stable and optimized Marshal serializer for Active Record models.

Can be enabled with `config.active_record.marshalling_format_version = 7.1`.

---

`ActiveRecord::Base.serialize` no longer uses YAML by default.

YAML isn't particularly performant and can lead to security issues  
if not used carefully.

Unfortunately there isn't really any good serializers in Ruby's stdlib  
to replace it.

The obvious choice would be JSON, which is a fine format for this use case,  
however the JSON serializer in Ruby's stdlib isn't strict enough, as it fallback  
to casting unknown types to strings, which could lead to corrupted data.

Some third party JSON libraries like `Oj` have a suitable strict mode.

So it's preferable that users choose a serializer based on their own constraints.

The original default can be restored by setting `config.active_record.default_column_serializer = YAML`.

---

Add support for exclusion constraints (PostgreSQL-only).

```ruby
add_exclusion_constraint :invoices, "daterange(start_date, end_date) WITH &&", using: :gist, name: "invoices_date_overlap"
remove_exclusion_constraint :invoices, name: "invoices_date_overlap"
```

See PostgreSQL's [`CREATE TABLE ... EXCLUDE ...`](https://www.postgresql.org/docs/12/sql-createtable.html#SQL-CREATETABLE-EXCLUDE) documentation for more on exclusion constraints.

## 7.1.1

Nothing to report.

## 7.1.2

**Active Support**

Prevent global cache options being overwritten when setting dynamic options  
inside a `ActiveSupport::Cache::Store#fetch` block.

---

Fix `ActiveSupport::Cache` to handle outdated Marshal payload from Rails 6.1 format.

Active Support's Cache is supposed to treat a Marshal payload that can no longer be  
deserialized as a cache miss. It fail to do so for compressed payload in the Rails 6.1  
legacy format.

**Action View**

Fix the `capture` view helper compatibility with HAML and Slim

When a blank string was captured in HAML or Slim (and possibly other template engines)  
it would instead return the entire buffer.

## 7.1.3

**Active Record**

Fix `load_async` to work with query cache.

## 7.1.4

**Active Record**

Lots of fixes lol.

**Railties**

Revert the use of `Concurrent.physical_processor_count` in default Puma config

While for many people this saves one config to set, for many others using  
a shared hosting solution, this cause the default configuration to spawn  
way more workers than reasonable.

There is unfortunately no reliable way to detect how many cores an application  
can realistically use, and even then, assuming the application should use  
all the machine resources is often wrong.

# Changes in 7.2.0 through 7.2.1

## 7.2.0

**Active Support**

Remove deprecated `#to_default_s` from `Array`, `Date`, `DateTime` and `Time`.

---

Remove deprecated `config.active_support.use_rfc4122_namespaced_uuids`.

---

Remove deprecated `config.active_support.disable_to_s_conversion`.

---

Remove deprecated support for `config.active_support.cache_format_version = 6.1`.

---

Use logical core count instead of physical core count to determine the  
default number of workers when parallelizing tests.

---

**Active Record**

Strict loading using `:n_plus_one_only` does not eagerly load child associations.

With this change, child associations are no longer eagerly loaded, to  
match intended behavior and to prevent non-deterministic order issues caused  
by calling methods like `first` or `last`. As `first` and `last` don't cause  
an N+1 by themselves, calling child associations will no longer raise.

---

When using a `DATABASE_URL`, allow for a configuration to map the protocol in the URL to a specific database  
adapter. This allows decoupling the adapter the application chooses to use from the database connection details  
set in the deployment environment.

**Action View**

Deprecate passing `nil` as value for the `model:` argument to the `form_with` method.

**Action Pack**

Add rate limiting API.

---

Add `racc` as a dependency since it will become a bundled gem in Ruby 3.4.0

**Railties**

Rails console now indicates application name and the current Rails environment

## 7.2.1

**Active Record**

Allow to eager load nested nil associations.