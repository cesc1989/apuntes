# Upgrade to Rails 7.1.4

# Breaking in CI

First attempt broke with these warnings:
```bash
DEPRECATION WARNING: DeprecatedConstantAccessor.deprecate_constant without a deprecator is deprecated (called from <main> at /app/config/application.rb:9)
DEPRECATION WARNING: Support for `config.active_support.cache_format_version = 6.1` has been deprecated and will be removed in Rails 7.2.

Check the Rails upgrade guide at https://guides.rubyonrails.org/upgrading_ruby_on_rails.html#new-activesupport-cache-serialization-format
for more information on how to upgrade.
 (called from <main> at /app/config/environment.rb:7)
```

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