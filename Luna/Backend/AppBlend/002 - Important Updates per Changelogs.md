# 6.1.6.1

See https://github.com/rails/rails/releases/tag/v6.1.6.1

## Active Record

Change ActiveRecord::Coders::YAMLColumn default to safe_load.

# 6.1.7

See https://github.com/rails/rails/releases/tag/v6.1.7

## Active Record

Symbol is allowed by default for YAML columns.

---

Fix `ActiveRecord::Store` to serialize as a regular Hash

Previously it would serialize as an `ActiveSupport::HashWithIndifferentAccess`  
which is wasteful and cause problem with YAML safe_load.

# 7.0.0

See https://github.com/rails/rails/releases/tag/v7.0.0

## Active Support

Deprecate passing a format to `#to_s` in favor of `#to_formatted_s` in `Array`, `Range`, `Date`, `DateTime`, `Time`,  
`BigDecimal`, `Float` and, `Integer`.

## Railtie

The setter `config.autoloader=` has been deleted. `zeitwerk` is the only  
available autoloading mode.

---

During initialization, you cannot autoload reloadable classes or modules  
like application models, unless they are wrapped in a `to_prepare` block.  
For example, from `config/initializers/*`, or in application, engines, or  
railties initializers.

> Please check the [autoloading  guide](https://guides.rubyonrails.org/v7.0/autoloading_and_reloading_constants.html#autoloading-when-the-application-boots) for details.

---

Fix compatibility with `psych >= 4`.

> Starting in Psych 4.0.0 `YAML.load` behaves like `YAML.safe_load`. To preserve compatibility  `Rails.application.config_for` now uses `YAML.unsafe_load` if available.