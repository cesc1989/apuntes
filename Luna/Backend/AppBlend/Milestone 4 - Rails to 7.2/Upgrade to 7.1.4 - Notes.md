# Upgrade to Rails 7.1.4

# Deprecation Warnings

## DeprecatedConstantAccessor

```bash
DEPRECATION WARNING: DeprecatedConstantAccessor.deprecate_constant without a deprecator is deprecated (called from <main> at /app/config/application.rb:9)
```

## cache_format_version

```bash
DEPRECATION WARNING: Support for `config.active_support.cache_format_version = 6.1` has been deprecated and will be removed in Rails 7.2.

Check the Rails upgrade guide at https://guides.rubyonrails.org/upgrading_ruby_on_rails.html#new-activesupport-cache-serialization-format
for more information on how to upgrade.
 (called from <main> at /app/config/environment.rb:7)
```

## application secrets

```bash
DEPRECATION WARNING: `Rails.application.secrets` is deprecated in favor of `Rails.application.credentials` and will be removed in Rails 7.2. (called from <top (required)> at /Users/francisco/projects/luna-project/backend/config/environment.rb:7)
```


# 🎉 undefined method _routes for Profile:Module 🎉

and this error:
```bash
bin/rails aborted!
NoMethodError: undefined method `_routes' for Profile:Module

            if !base._routes.equal?(@_proxy._routes)
                    ^^^^^^^^
/app/app/models/concerns/profile.rb:5:in `include'
/app/app/models/concerns/profile.rb:5:in `<module:Profile>'
/app/app/models/concerns/profile.rb:3:in `<main>'
/app/app/models/patient.rb:29:in `<class:Patient>'
/app/app/models/patient.rb:4:in `<main>'
/app/app/admin/concierge/unverified_patients_report.rb:3:in `<main>'
/app/config/routes.rb:92:in `block in <main>'
/app/config/routes.rb:6:in `<main>'
/app/config/environment.rb:7:in `<main>'
Tasks: TOP => environment
(See full trace by running task with --trace)
```

also happens in local tests:
```bash
An error occurred while loading rails_helper.
Failure/Error: include Rails.application.routes.url_helpers

NoMethodError:
  undefined method `_routes' for Profile:Module

              if !base._routes.equal?(@_proxy._routes)
                      ^^^^^^^^
# /Users/francisco/.gem/ruby/3.1.0/gems/actionpack-7.1.0/lib/action_dispatch/routing/route_set.rb:606:in `included'
# ./app/models/concerns/profile.rb:5:in `include'
# ./app/models/concerns/profile.rb:5:in `<module:Profile>'
# ./app/models/concerns/profile.rb:3:in `<top (required)>'
```

This is fixed in Rails 7.1.3. See [Action Pack](https://github.com/rails/rails/releases/tag/v7.1.3):
```
- Fix including `Rails.application.routes.url_helpers` directly in an  
    `ActiveSupport::Concern.`
    
    Jonathan Hefner
```

# You tried to define an enum named "status" on the model but this will generate a instance method "frozen?", which is already defined by Active Record

This error:
```bash
An error occurred while loading rails_helper.
Failure/Error:
  enum status: {
    # No invoicing attempts have been made; target changes are allowed
    draft: 0,
    # No invoicing attempts have been made; target changes are NOT allowed
    frozen: 1,
    # We are about to attempt a charge or a charge is being attempted in Stripe (retrying)
    processing: 5,
    # Payment was successful
    succeeded: 10,
    # Manually aborted by a human operator or automatically aborted by the system due to changing circumstances

ArgumentError:
  You tried to define an enum named "status" on the model "InvoicingTarget", but this will generate a instance method "frozen?", which is already defined by Active Record.
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.1.3/lib/active_record/enum.rb:385:in `raise_conflict_error'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.1.3/lib/active_record/enum.rb:378:in `detect_enum_conflict!'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.1.3/lib/active_record/enum.rb:308:in `define_enum_methods'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.1.3/lib/active_record/enum.rb:276:in `block (2 levels) in _enum'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.1.3/lib/active_record/enum.rb:270:in `each_pair'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.1.3/lib/active_record/enum.rb:270:in `each'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.1.3/lib/active_record/enum.rb:270:in `block in _enum'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.1.3/lib/active_record/enum.rb:260:in `module_eval'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.1.3/lib/active_record/enum.rb:260:in `_enum'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.1.3/lib/active_record/enum.rb:225:in `block in enum'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.1.3/lib/active_record/enum.rb:225:in `each'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.1.3/lib/active_record/enum.rb:225:in `enum'
# /Users/francisco/.gem/ruby/3.1.0/gems/stateful_enum-0.7.0/lib/stateful_enum/active_record_extension.rb:14:in `enum'
# ./app/models/invoicing_target.rb:17:in `<class:InvoicingTarget>'
# ./app/models/invoicing_target.rb:4:in `<top (required)>'
```

This error happens in the `InvoicingTarget` model because it defines a `frozen` status which is not valid.
```ruby
enum status: {
    # No invoicing attempts have been made; target changes are allowed
    draft: 0,
    # No invoicing attempts have been made; target changes are NOT allowed
    frozen: 1,
```

This is because `stateful_enum` generates methods from the keys. So it will generate a `frozen?` method which is [already defined by ActiveRecord](https://apidock.com/rails/v7.0.0/ActiveRecord/Core/ClassMethods/frozen%3F)

See also similar issue report -> https://github.com/amatsuda/stateful_enum/issues/25
