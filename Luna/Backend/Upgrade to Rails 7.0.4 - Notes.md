# Upgrade to Rails 7.0.4 from 6.1.7 - Notes

[RailsDiff](https://railsdiff.org/6.1.7/7.0.4) shows multiple changes but due to how custom Edge is I don't consider good to apply them to this codebase.

Also, after running `bundle exec rails app:update` I left everything untouched because this app has a lot of years running and there's a lot of things that are better left untouched.

## Updated to 6.1.7

Edge sits at Rails 6.1.6. First step to take it to version 7.0.4 was to updated it to latest minor patch/security patch. After updating it to 6.1.7 got this error in ALL tests.

### Psych::DisallowedClass:

```bash
Psych::DisallowedClass:
       Tried to load unspecified class: BigDecimal
     # (eval):2:in `big_decimal'
     # /Users/francisco/.gem/ruby/3.1.0/bundler/gems/audited-288097842eac/lib/audited/audit.rb:20:in `load'
     # /Users/francisco/.gem/ruby/3.1.0/bundler/gems/audited-288097842eac/lib/audited/auditor.rb:268:in `block in write_audit'
     # /Users/francisco/.gem/ruby/3.1.0/bundler/gems/audited-288097842eac/lib/audited/auditor.rb:267:in `write_audit'
     # /Users/francisco/.gem/ruby/3.1.0/bundler/gems/audited-288097842eac/lib/audited/auditor.rb:246:in `audit_create'
```

```bash
Psych::DisallowedClass:
Tried to load unspecified class: ActiveSupport::HashWithIndifferentAccess
# /usr/local/bundle/bundler/gems/audited-288097842eac/lib/audited/audit.rb:20:in `load'
# /usr/local/bundle/bundler/gems/audited-288097842eac/lib/audited/auditor.rb:268:in `block in write_audit'
# /usr/local/bundle/bundler/gems/audited-288097842eac/lib/audited/auditor.rb:267:in `write_audit'
# /usr/local/bundle/bundler/gems/audited-288097842eac/lib/audited/auditor.rb:252:in `audit_update'
```

There are two solutions in this Stack Overflow question.

One is listing every class in `config/application.rb`
```ruby
config.active_record.yaml_column_permitted_classes = []
```

i.e:
```ruby
config.active_record.yaml_column_permitted_classes = [Symbol, Hash, Array, ActiveSupport::HashWithIndifferentAccess]
```

But it was breaking for a whole lot of classes in Backend so I don't see this a good alternative.

I decided instead to try this
```ruby
config.active_record.use_yaml_unsafe_load = true
```

And now my local tests passed.

The explanation to why this happens is in [this Discuss post](https://discuss.rubyonrails.org/t/cve-2022-32224-possible-rce-escalation-bug-with-serialized-columns-in-active-record/81017).

The summary is that the second option above is insecure and the first one is the way to go.

## Gem devise_token_auth mismatch

Current version 1.2.0 supports rails version until 6.2.0

```
Could not find compatible versions

Because devise_token_auth >= 1.1.5, < 1.2.1 depends on rails >= 4.2.0, < 6.2
  and Gemfile depends on devise_token_auth = 1.2.0,
  rails >= 4.2.0, < 6.2 is required.
So, because Gemfile depends on rails = 7.0.0,
  version solving has failed.
```

Changed it to version 1.2.1 which is the [minimum version](https://github.com/lynndylanhurley/devise_token_auth/blob/v1.2.1/devise_token_auth.gemspec#L25) requiring Rails < 7.1

## Gem lol_dba mismatch

Same as above with this gem:
```
Could not find compatible versions

Because lol_dba >= 2.2.0, < 2.4.0 depends on railties >= 3.0, < 7.0
  and rails >= 7.0.0, < 7.0.1 depends on railties = 7.0.0,
  lol_dba >= 2.2.0, < 2.4.0 is incompatible with rails >= 7.0.0, < 7.0.1.
So, because Gemfile depends on rails = 7.0.0
  and Gemfile depends on lol_dba = 2.2.0,
  version solving has failed.
```

Changed the version to 2.3.0 but it says it cannot be found??:
```
Could not find gem 'lol_dba (= 2.3.0)' in rubygems repository https://rubygems.org/ or installed locally.

The source contains the following gems matching 'lol_dba':
  * lol_dba-1.0
  * lol_dba-2.2.0
  * lol_dba-2.4.0
```

And it's right. That version is not [available in rubygems](https://rubygems.org/gems/lol_dba/versions):
![[Pasted image 20240729105042.png]]

Changed it to version 2.4.0

## Gem acts-as-taggable-on mismatch

```
Could not find compatible versions

Because acts-as-taggable-on >= 7.0.0, < 9.0.0 depends on activerecord >= 5.0, < 6.2
  and rails >= 7.0.0, < 7.0.1 depends on activerecord = 7.0.0,
  acts-as-taggable-on >= 7.0.0, < 9.0.0 is incompatible with rails >= 7.0.0, < 7.0.1.
So, because Gemfile depends on acts-as-taggable-on ~> 7
  and Gemfile depends on rails = 7.0.0,
  version solving has failed.
```

Had to changed it to [version 9.0.0](https://github.com/mbleigh/acts-as-taggable-on/blob/v9.0.0/acts-as-taggable-on.gemspec#L25).

## Gem audited mismatch

This sound very bad:
```
Could not find compatible versions

Because every version of audited depends on activerecord >= 4.2, <= 6.2
  and rails >= 7.0.0, < 7.0.1 depends on activerecord = 7.0.0,
  every version of audited is incompatible with rails >= 7.0.0, < 7.0.1.
So, because Gemfile depends on audited >= 0
  and Gemfile depends on rails = 7.0.0,
  version solving has failed.
```

I'd need to point to [version 5.1.0](https://github.com/collectiveidea/audited/blob/v5.1.0/audited.gemspec) but in the backend repo this gem was forked and it's fixed to an old version.

Changed it to point to the source instead of the fork.

## Gem paranoia mismatch

```
Could not find compatible versions

Because paranoia >= 2.4.3, < 2.5.0 depends on activerecord >= 4.0, < 6.2
  and rails >= 7.0.0, < 7.0.1 depends on activerecord = 7.0.0,
  paranoia >= 2.4.3, < 2.5.0 is incompatible with rails >= 7.0.0, < 7.0.1.
So, because Gemfile depends on paranoia = 2.4.3
  and Gemfile depends on rails = 7.0.0,
  version solving has failed.
```

Update it to [version 2.5.0](https://github.com/rubysherpas/paranoia/blob/v2.5.0/paranoia.gemspec).

## Gem active_record-postgres-constraints mismatch

```
Could not find compatible versions

Because every version of active_record-postgres-constraints depends on rails >= 5.0, <= 7.0
  and Gemfile depends on active_record-postgres-constraints >= 0,
  rails >= 5.0, <= 7.0 is required.
So, because Gemfile depends on rails = 7.0.1,
  version solving has failed.
```

This gem is also forked.

The issue here is that bundler is not pulling latest commit from the fork. In the fork these are the last commits:

- Merge pull request #2 from lunacare/afh-rails (2022)
- Merge pull request #1 from lunacare/afh-where (2021)

But when opening the gem in the installation folder it only shows the first commit:
```bash
in ~/.gem/ruby/3.1.0/bundler/gems/active_record-postgres-constraints-ff0622005ad4 on master [!]
$ glone -5
ff06220 (grafted, HEAD -> master) Merge pull request #1 from lunacare/afh-where
```

Why? Don't know but solve by pointing it to the commit instead of the branch:

```diff
-gem "active_record-postgres-constraints", github: "lunacare/active_record-postgres-constraints", branch: "master"
+gem "active_record-postgres-constraints", github: "lunacare/active_record-postgres-constraints", ref: "8427886"
```

That way bundler downloaded the actual latest changes and I saw it reflected in the gems folder by seeing two different versions of the gem:
```bash
~/.gem/ruby/3.1.0/bundler/gems
$ ll
total 0
Jul 30 15:24 active_record-postgres-constraints-842788678eba
Jul  9 17:55 active_record-postgres-constraints-ff0622005ad4
```

Bundler docs about git as source -> https://bundler.io/guides/git.html


# CI Errors

## Rails::Engine is abstract, you cannot instantiate it directly. (RuntimeError)

Got this error when building the release image in the CI:
```bash
/usr/local/bundle/gems/railties-7.0.0/lib/rails/railtie.rb:246:in `initialize': Rails::Engine is abstract, you cannot instantiate it directly. (RuntimeError)
	from /usr/local/bundle/gems/railties-7.0.0/lib/rails/railtie.rb:184:in `new'
	from /usr/local/bundle/gems/railties-7.0.0/lib/rails/railtie.rb:184:in `instance'
	from /usr/local/bundle/gems/railties-7.0.0/lib/rails/railtie.rb:223:in `method_missing'
	from /usr/local/bundle/gems/activesupport-7.0.0/lib/active_support/descendants_tracker.rb:90:in `descendants'
	from /usr/local/bundle/gems/activesupport-7.0.0/lib/active_support/callbacks.rb:923:in `block in define_callbacks'
	from /usr/local/bundle/gems/activesupport-7.0.0/lib/active_support/callbacks.rb:920:in `each'
	from /usr/local/bundle/gems/activesupport-7.0.0/lib/active_support/callbacks.rb:920:in `define_callbacks'
	from /usr/local/bundle/gems/railties-7.0.0/lib/rails/engine.rb:427:in `<class:Engine>'
	from /usr/local/bundle/gems/railties-7.0.0/lib/rails/engine.rb:349:in `<module:Rails>'
	from /usr/local/bundle/gems/railties-7.0.0/lib/rails/engine.rb:11:in `<top (required)>'
	from /usr/local/bundle/gems/railties-7.0.0/lib/rails/application.rb:11:in `require'
	from /usr/local/bundle/gems/railties-7.0.0/lib/rails/application.rb:11:in `<top (required)>'
	from /usr/local/bundle/gems/railties-7.0.0/lib/rails.rb:13:in `require'
	from /usr/local/bundle/gems/railties-7.0.0/lib/rails.rb:13:in `<top (required)>'
	from /app/lib/lunacare/environment.rb:3:in `require'
	from /app/lib/lunacare/environment.rb:3:in `<top (required)>'
	from /app/lib/lunacare.rb:3:in `require_relative'
	from /app/lib/lunacare.rb:3:in `<top (required)>'
	from /app/config/luna.rb:3:in `require_relative'
	from /app/config/luna.rb:3:in `<top (required)>'
	from /app/config/boot.rb:5:in `require_relative'
	from /app/config/boot.rb:5:in `<top (required)>'
	from bin/rails:4:in `require_relative'
	from bin/rails:4:in `<main>'
```

This was already seen at [[Upgrade Ruby to 3.1.0]] the fix is to use Rails 7.0.1

## undefined method reference for ActiveSupport::Dependencies:Module

This is a Devise related error.

```bash
NoMethodError: undefined method `reference' for ActiveSupport::Dependencies:Module

    ActiveSupport::Dependencies.reference(arg)
                               ^^^^^^^^^^
/usr/local/bundle/gems/devise-4.7.3/lib/devise.rb:321:in `ref'
/usr/local/bundle/gems/devise-4.7.3/lib/devise.rb:340:in `mailer='
/usr/local/bundle/gems/devise-4.7.3/lib/devise.rb:342:in `<module:Devise>'
/usr/local/bundle/gems/devise-4.7.3/lib/devise.rb:11:in `<main>'
/usr/local/bundle/gems/bootsnap-1.10.3/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:30:in `require'
/usr/local/bundle/gems/bootsnap-1.10.3/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:30:in `require'
/usr/local/bundle/gems/bundler-2.4.22/lib/bundler/runtime.rb:60:in `block (2 levels) in require'
/usr/local/bundle/gems/bundler-2.4.22/lib/bundler/runtime.rb:55:in `each'
/usr/local/bundle/gems/bundler-2.4.22/lib/bundler/runtime.rb:55:in `block in require'
/usr/local/bundle/gems/bundler-2.4.22/lib/bundler/runtime.rb:44:in `each'
/usr/local/bundle/gems/bundler-2.4.22/lib/bundler/runtime.rb:44:in `require'
/usr/local/bundle/gems/bundler-2.4.22/lib/bundler.rb:187:in `require'
/app/config/application.rb:9:in `<main>'
```

Solution is to upgrade Devise to [version 4.8.1](https://github.com/heartcombo/devise/pull/5357#issuecomment-995863195)

## NoMethodError: undefined method use_yaml_unsafe_load=' for ActiveRecord::Base:Class

New error:
```bash
rails aborted!
NoMethodError: undefined method `use_yaml_unsafe_load=' for ActiveRecord::Base:Class
/app/config/environment.rb:7:in `<main>'
```

The CI threw this error when building for rails 7.0.1. Updated it to Rails 7.0.4.

## Error with class being loaded in config/database.yml

This is error:
```bash
NameError: Cannot load database configuration:
uninitialized constant Runtime
Did you mean?  RuntimeError
/app/config/database.yml:4:in `<main>'
/app/config/environment.rb:7:in `<main>'
```

Also happens when running tests locally:
```bash
An error occurred while loading rails_helper.
Failure/Error: require File.expand_path("../../config/environment", __FILE__)

NameError:
  Cannot load database configuration:
  uninitialized constant Runtime
  Did you mean?  RuntimeError
# ./config/database.yml:4:in `<main>'
# /Users/francisco/.gem/ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/configuration_file.rb:48:in `render'
# /Users/francisco/.gem/ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/configuration_file.rb:22:in `parse'
# /Users/francisco/.gem/ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/configuration_file.rb:18:in `parse'
# /Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.4/lib/rails/application/configuration.rb:335:in `database_configuration'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/railtie.rb:266:in `block (2 levels) in <class:Railtie>'
# /Users/francisco/.gem/ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/lazy_load_hooks.rb:95:in `class_eval'
# /Users/francisco/.gem/ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/lazy_load_hooks.rb:95:in `block in execute_hook'
# /Users/francisco/.gem/ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/lazy_load_hooks.rb:85:in `with_execution_control'
# /Users/francisco/.gem/ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/lazy_load_hooks.rb:90:in `execute_hook'
# /Users/francisco/.gem/ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/lazy_load_hooks.rb:60:in `block in on_load'
# /Users/francisco/.gem/ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/lazy_load_hooks.rb:59:in `each'
# /Users/francisco/.gem/ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/lazy_load_hooks.rb:59:in `on_load'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/railtie.rb:262:in `block in <class:Railtie>'
# /Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.4/lib/rails/initializable.rb:32:in `instance_exec'
# /Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.4/lib/rails/initializable.rb:32:in `run'
# /Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.4/lib/rails/initializable.rb:61:in `block in run_initializers'
# /Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.4/lib/rails/initializable.rb:60:in `run_initializers'
# /Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.4/lib/rails/application.rb:372:in `initialize!'
# ./config/environment.rb:7:in `<top (required)>'
# ./spec/rails_helper.rb:8:in `require'
# ./spec/rails_helper.rb:8:in `<top (required)>'
```

The issue is because in the file `config/database.yml` the class Runtime is being loaded:
```yml
defaults: &defaults
  adapter: postgresql
  variables:
    <% if Runtime.rails_server? %>
```

Is this related to the `use_yaml_unsafe_load` method used in application.rb?
```ruby
module LunaApi
  class Application < Rails::Application
    config.active_record.use_yaml_unsafe_load = true
  end
end
```

Probably it is but fixed by loading this class in `config/application.rb`
```ruby
require_relative "../app/lib/runtime"
```

#  undefined method connection_config' for ActiveRecord::Base:Class

Getting this:
```
NoMethodError:
  undefined method `connection_config' for ActiveRecord::Base:Class
  Did you mean?  connection_db_config
                 connection_pool
                 connection_class
                 connection_class=
                 connection_class?
```

By reading in [Stack Overflow](https://stackoverflow.com/questions/65459059/connection-config-deprecated-warning-with-rspec-after-migrating-to-rails-6-1) looks like this error comes from an outdated gem. In this case the trace points to `active_record-postgres-constraints`:

```
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/dynamic_matchers.rb:22:in `method_missing'
# /Users/francisco/.gem/ruby/3.1.0/bundler/gems/active_record-postgres-constraints-842788678eba/lib/active_record/postgres/constraints/railtie.rb:30:in `pg?'
# /Users/francisco/.gem/ruby/3.1.0/bundler/gems/active_record-postgres-constraints-842788678eba/lib/active_record/postgres/constraints/railtie.rb:13:in `block (2 levels) in <class:Railtie>'
# /Users/francisco/.gem/ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/lazy_load_hooks.rb:95:in `class_eval'
```

And when inspecting the forked repo I can see the error:
```ruby
def pg?
  config = ActiveRecord::Base.connection_config
```

Decided to test by opening the downloaded gem and doing a local edit:
```ruby
if defined?(::Rails::Railtie)
  module ActiveRecord
    module Postgres
      module Constraints
        class Railtie < ::Rails::Railtie
          # (...)

          def pg?
            config = ActiveRecord::Base.connection_db_config
            return true if config && config[:adapter].in?(%w[postgresql postgis])

			# (...)
            false
          end
        end
      end
    end
  end
end
```

But got this error:
```
NoMethodError:
  undefined method `[]' for #<ActiveRecord::DatabaseConfigurations::HashConfig:0x00000001089cb9a0 @env_name="test", @name="primary", @configuration_hash={:adapter=>"postgresql", :variables=>{"statement_timeout"=>0}, :template=>"template0", :host=>"localhost", :username=>"francisco", :password=>nil, :database=>"luna_api_test", :min_messages=>"warning"}>

              return true if config && config[:adapter].in?(%w[postgresql postgis])
```

When inspecting the instance in the error:
```ruby
<ActiveRecord::DatabaseConfigurations::HashConfig:0x00000001089cb9a0
@env_name="test",
@name="primary",
@configuration_hash={
	:adapter=>"postgresql",
	:variables=>{"statement_timeout"=>0},
	:template=>"template0",
	:host=>"localhost",
	:username=>"francisco",
	:password=>nil,
	:database=>"luna_api_test",
	:min_messages=>"warning"
}>
```

Now it needs to access the hash by accessing an method of the instance:
```ruby
config = ActiveRecord::Base.connection_db_config
return true if config && config.configuration_hash[:adapter].in?(%w[postgresql postgis])
```


#  uninitialized constant EmailLogHubspot

In config/initializers/action_mailer:
```ruby
ActionMailer::Base.register_observer(EmailLogHubspot)
```

but it's causing this error:
```bash
NameError:
  uninitialized constant EmailLogHubspot

  ActionMailer::Base.register_observer(EmailLogHubspot)
                                       ^^^^^^^^^^^^^^^
# ./config/initializers/action_mailer.rb:3:in `<top (required)>'
# /Users/francisco/.gem/ruby/3.1.0/gems/bootsnap-1.10.3/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:48:in `load'
# /Users/francisco/.gem/ruby/3.1.0/gems/bootsnap-1.10.3/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:48:in `load'
```

Do I need to also require it?

Fixed it by wrapping the initializer into some custom Rails code
```ruby
Rails.application.config.after_initialize do
  ActionMailer::Base.register_observer(EmailLogHubspot)
end
```

As [explained here](https://guides.rubyonrails.org/v7.0/autoloading_and_reloading_constants.html#autoloading-when-the-application-boots) and mentioned [here](https://stackoverflow.com/a/73463696/1407371) by Xavier Noira.

# Zeitwerk

I had to wrap lots of code in initializers folder with this block:
```ruby
Rails.application.config.after_initialize do
end
```

Because Zeitwerk changed the way code is loaded and as [Xavier Noira said](https://stackoverflow.com/a/73463720/1407371):
> This is unrelated to Zeitwerk, autoloading from initializers was just wrong conceptually regardless of the autoloader.

