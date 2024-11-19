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


# undefined method _routes for Profile:Module

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