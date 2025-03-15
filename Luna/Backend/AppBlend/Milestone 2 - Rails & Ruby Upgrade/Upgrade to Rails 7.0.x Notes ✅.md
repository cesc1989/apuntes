# Upgrade to Rails 7.0.? Notes ✅

[RailsDiff](https://railsdiff.org/6.1.7/7.0.4) shows multiple changes but due to how custom Edge is I don't consider good to apply them to this codebase.

Also, after running `bundle exec rails app:update` I left everything untouched because this app has a lot of years running and there's a lot of things that are better left untouched.

# Updated to 6.1.6.1 ✅

Edge sits at Rails 6.1.6. First step to take it to version 7.0.4 was to updated it to latest minor patch/security patch. After updating it to 6.1.6.1 got this error in ALL tests.

## Psych::DisallowedClass ✅

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

## Gem devise_token_auth mismatch ✅

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

## Gem lol_dba mismatch ✅

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

![[lol_dba_versions.png]]

Changed it to version 2.4.0

## Gem acts-as-taggable-on mismatch ✅

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

## Gem audited mismatch ✅

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

## Gem paranoia mismatch ✅

```
Could not find compatible versions

Because paranoia >= 2.4.3, < 2.5.0 depends on activerecord >= 4.0, < 6.2
  and rails >= 7.0.0, < 7.0.1 depends on activerecord = 7.0.0,
  paranoia >= 2.4.3, < 2.5.0 is incompatible with rails >= 7.0.0, < 7.0.1.
So, because Gemfile depends on paranoia = 2.4.3
  and Gemfile depends on rails = 7.0.0,
  version solving has failed.
```

Update it to [version 2.5.0](https://github.com/rubysherpas/paranoia/blob/v2.5.0/paranoia.gemspec#L27) - [Release](https://github.com/rubysherpas/paranoia/releases/tag/v2.5.0)

## Gem active_record-postgres-constraints mismatch ✅

```
Could not find compatible versions

Because every version of active_record-postgres-constraints depends on rails >= 5.0, <= 7.0
  and Gemfile depends on active_record-postgres-constraints >= 0,
  rails >= 5.0, <= 7.0 is required.
So, because Gemfile depends on rails = 7.0.1,
  version solving has failed.
```

> This gem is also forked.

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

# Rails::Engine is abstract, you cannot instantiate it directly. (RuntimeError) - Rails 7.0.0 Error ✅

Got this error when building the release image in the CI:
```bash
/usr/local/bundle/gems/railties-7.0.0/lib/rails/railtie.rb:246:in `initialize': Rails::Engine is abstract, you cannot instantiate it directly. (RuntimeError)
	from /usr/local/bundle/gems/railties-7.0.0/lib/rails/railtie.rb:184:in `new'
	from /usr/local/bundle/gems/railties-7.0.0/lib/rails/railtie.rb:184:in `instance'
	from /usr/local/bundle/gems/railties-7.0.0/lib/rails/railtie.rb:223:in `method_missing'
	from /usr/local/bundle/gems/activesupport-7.0.0/lib/active_support/descendants_tracker.rb:90:in `descendants'
```

This was already seen at [[Upgrade Ruby to 3.1.0]] ==the fix is to use Rails 7.0.1==.

# 👉🏽 undefined method reference for ActiveSupport::Dependencies:Module 👈🏽 - devise gem ✅

This is a Devise related error.

```bash
NoMethodError: undefined method reference for ActiveSupport::Dependencies:Module

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

# NoMethodError: undefined method use_yaml_unsafe_load=' for ActiveRecord::Base:Class - Rails 7.0.1 Error ✅

New error:
```bash
rails aborted!
NoMethodError: undefined method `use_yaml_unsafe_load=' for ActiveRecord::Base:Class
/app/config/environment.rb:7:in `<main>'
```

The CI threw this error when building for rails 7.0.1. ==Fixed by updating to Rails 7.0.4==.

Also got this error when running the `rails app:update` command:
```bash
    conflict  bin/setup
Overwrite /Users/francisco/projects/luna-project/backend/bin/setup? (enter "h" for help) [Ynaqdhm] n
        skip  bin/setup
       rails  active_storage:update
rails aborted!
NoMethodError: undefined method `use_yaml_unsafe_load=' for ActiveRecord::Base:Class
/Users/francisco/projects/luna-project/backend/config/environment.rb:7:in `<main>'
Tasks: TOP => active_storage:update => environment
(See full trace by running task with --trace)
```

# Error with class being loaded in config/database.yml ✅

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

#  undefined method connection_config' for ActiveRecord::Base:Class ✅ - gema active_record-postgres-constraints

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

# Rails assets:precompile

Got this error in the CI and local env:
```bash
$ RAILS_ENV=production be rails assets:precompile --trace

rails aborted!
NoMethodError: private method `warn' called for nil:NilClass

        ::Rails.logger.warn "Application uninitialized: Try calling YourApp::Application.initialize!"
                      ^^^^^
/Users/francisco/.gem/ruby/3.1.0/gems/sprockets-rails-3.5.1/lib/sprockets/railtie.rb:185:in `build_environment'
/Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.4/lib/rails/railtie.rb:226:in `public_send'
/Users/francisco/.gem/ruby/3.1.0/gems/railties-7.0.4/lib/rails/railtie.rb:226:in `method_missing'
/Users/francisco/.gem/ruby/3.1.0/gems/sprockets-rails-3.5.1/lib/sprockets/rails/task.rb:20:in `environment'
```

The solution was to add this line in `config/application.rb`:
```ruby
Rails.logger ||= Logger.new(STDOUT)
```

## webpack not found ✅

Also caught in the CI:
```bash
Compiling...
Compilation failed:
yarn run v1.22.17
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.

error Command "webpack" not found.
```

The solution that worked so far was [Michael Hoste](https://github.com/rails/rails/issues/43906#issuecomment-1099992310) suggests:
```ruby
# lib/tasks/yarn.rake

Rake::Task["assets:precompile"].enhance ["yarn:install"]
```

# Error with rubocop gems

This error:
```
Downloading rubocop-rails-2.15.2 revealed dependencies not in the API or the lockfile (rubocop (< 2.0, >= 1.7.0)). Running `bundle update rubocop-rails` should fix the problem.
```

Ran the command suggested in the error and it downgraded some gems. Looks like something 
change some time ago when doing other commands.

# undefined method silence for Logger ✅

This error pop up in the CI for lots of tests:
```bash
NoMethodError:
	undefined method silence for #<Logger:0x00007f82ed94f6c0 @level=0, @progname=nil, @default_formatter=#<Logger::Formatter:0x00007f82ed94f300 @datetime_format=nil>, @formatter=nil, @logdev=#<Logger::LogDevice:0x00007f82ed94ef68 @shift_period_suffix=nil, @shift_size=nil, @shift_age=nil, @filename=nil, @dev=#<IO:<STDOUT>>, @binmode=false, @mon_data=#<Monitor:0x00007f82ed94ee50>, @mon_data_owner_object_id=23660>, @level_override={}>

logger.silence do
```

Turns out it's related to the gem `activerecord-session_store` when it's updated to version 2.0 and up. In this case it's on 2.1.0

In [this issue](https://github.com/rails/activerecord-session_store/issues/176) they discussed this error and also provided a fix.

The fix is to put this line:
```ruby
Rails.logger.class.include ActiveSupport::LoggerSilence
```

Where? I put it in `config/application.rb`. In [this comment](https://github.com/rails/activerecord-session_store/issues/176#issuecomment-797665880), Swanson suggests to add it in `config/initializers/session_store.rb`.

Turns out it's better in `config/environments/test.rb` as per [[Pruebas de Rails 7#Pruebas que se rompen sin la configuración de LoggerSilence]]

# undefined method user_scopes for DocumentTag:Class ✅  - stateful_enum gem

```bash
An error occurred while loading rails_helper.
Failure/Error:
  DocumentTag.user_scopes.each_key do |scope|
    trait "for_#{scope}".to_sym do
      user_scope { scope }
    end
  end

NoMethodError:
  undefined method `user_scopes' for DocumentTag:Class
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/dynamic_matchers.rb:22:in `method_missing'
# ./spec/factories/document_tags.rb:11:in `block in <top (required)>'
# /Users/francisco/.gem/ruby/3.1.0/gems/factory_bot-6.4.6/lib/factory_bot/syntax/default.rb:37:in `instance_eval'
# /Users/francisco/.gem/ruby/3.1.0/gems/factory_bot-6.4.6/lib/factory_bot/syntax/default.rb:37:in `run'
# /Users/francisco/.gem/ruby/3.1.0/gems/factory_bot-6.4.6/lib/factory_bot/syntax/default.rb:7:in `define'
```

This is solved by updating [stateful_enum](https://github.com/amatsuda/stateful_enum) gem to 0.7.0

# Audited gem issues ✅

## undefined method last for 0:Integer in audited_changes method call ✅

Test run this appears `pruebas ./spec/models/chart_spec.rb:105`.

```bash
Failure/Error: audit.audited_changes["state"]&.last == "signed"

     NoMethodError:
       undefined method `last' for 0:Integer

                                     audit.audited_changes["state"]&.last == "signed"
                                                                   ^^^^^^
     # ./app/models/chart.rb:611:in `block in set_signature_date'
     # ./app/models/chart.rb:609:in `set_signature_date'
     # /Users/francisco/.gem/ruby/3.1.0/gems/stateful_enum-0.7.0/lib/stateful_enum/machine.rb:61:in `call'
     # /Users/francisco/.gem/ruby/3.1.0/gems/stateful_enum-0.7.0/lib/stateful_enum/machine.rb:61:in `block (3 levels) in initialize'
```

So we have two things:

1. the error happens in the calle to `audited_changes`
	1. The gem audited it's the version 5.1.0
	2. After updating to version 5.7.0, this error still happens
2. The stack trace points to gem `stateful_enum`
	1. This gem is on the latest version: 0.7.0

However, in omega branch stateful_enum is not on version 0.7.0 but on 0.6.0. ~~After changing the version back to 0.6.0 the error goes away... But another one creeps out~~.

Turns out the previous error happening in stateful_enum 0.6.0 gets fixed by updating the gem to version 0.7.0.

I created a replication app, setup a similar model with the same enum and after installing stateful_enum gem got the error about `undefined method user_scopes for DocumentTag:Class`. Once updated to latest version the error got away.

Now, I have to replicate the model that leads to _this_ error to see if it also happens because of this gem.

Gems mentioned
- [stateful_enum](https://github.com/amatsuda/stateful_enum)
- [audited](https://github.com/collectiveidea/audited)

**Solution**

The error was present in the audited gem. The project uses a forked version that was left in the version 4.9.0. Turns out, this code:
```ruby
audit.audited_changes["state"]&.last == "signed"
```

depends on audited storing enum values as strings instead of the underlying values (in this case, integers).

Also turns out [they changed the way audited stores enum values](https://github.com/collectiveidea/audited?tab=readme-ov-file#enum-storage) from version 4.10.
> In 4.10, the default behavior for enums changed from storing the value synthesized by Rails to the value stored in the DB. You can restore the previous behavior by setting the store_synthesized_enums configuration value:

The fix was to add this line to the audit initializer:
```ruby
Audited.store_synthesized_enums = true
```

## undefined method as_user for "Audited::Audit":String ✅

```bash
Failure/Error:
       Audited.audit_class.as_user(system_user.id) do
         last_audit.undo
       end

     NoMethodError:
       undefined method as_user for "Audited::Audit":String

             Audited.audit_class.as_user(system_user.id) do
                                ^^^^^^^^
     # ./app/workers/discharge_inactive_care_plan_worker.rb:25:in `perform'
     # ./spec/workers/discharge_inactive_care_plan_worker_spec.rb:51:in `block (4 levels) in <top (required)>'
```

Related issue reports:

- [Uninitialized Constant when defining an audit_class in the initializer](https://github.com/collectiveidea/audited/issues/608)
- [Use string class name and constantize](https://github.com/collectiveidea/audited/pull/609)

The fix is to update audited to 5.2.0.

# undefined method new_record? for []:Array - batch-loader & has_many_inversing # ✅

Test run this error appears: `pruebas ./spec/requests/graphql/mutations/scheduling/bulk_add_appointment_spec.rb:106`.

```bash
Failure/Error: association.target = records

     NoMethodError:
       undefined method new_record? for []:Array

           __sync!.public_send(method_name, *args, **opts, &block)
                  ^^^^^^^^^^^^

     # /Users/francisco/.gem/ruby/3.1.0/bundler/gems/batch-loader-350767424460/lib/batch_loader.rb:73:in public_send
     # /Users/francisco/.gem/ruby/3.1.0/bundler/gems/batch-loader-350767424460/lib/batch_loader.rb:73:in method_missing
     #   ./app/lib/active_record_extensions/query_methods/with_load_methods.rb:10:in load_many
     # ./app/graphql/types/appointment.rb:203:in block (2 levels) in care_plan
     # ./app/graphql/types/appointment.rb:197:in each
     # ./app/graphql/types/appointment.rb:197:in block in care_plan
```

Related to gem [batch-loader](https://github.com/exAspArk/batch-loader). There's a [fork](https://github.com/lunacare/batch-loader) in use and it's very outdated.

The stack trace points us to batch-loader gem but the error can be traced to previous steps.

It starts here:
```ruby
# app/graphql/types/care_plan.rb:604

BatchLoader::GraphQL.for(object).batch do |care_plans, loader|
  # (...)

  care_plans.each do |care_plan|
    appointments = Appointment.lazy_group(care_plan.id, :episode_id, ordering: { scheduled_date: :asc })
    exercise_program = ExerciseProgram.lazy(care_plan.id, :episode_id)

    care_plan
      .load_many(appointments, :appointments)
      .load_one(exercise_program, :exercise_program)
  end

  # (...)
end
```

and then the trace leads us to `/app/lib/active_record_extensions/query_methods/with_load_methods.rb:10:in load_many`:

```ruby
module ActiveRecordExtensions
  module QueryMethods
    module WithLoadMethods
      def load_many(records, association_name)
        association = self.association(association_name)
        association.loaded!
        association.target = records
        records.each { |record| association.set_inverse_instance(record) }
        self
      end

      def load_one(record, association_name)
        # (...)
      end
    end
  end
end
```

Notice the start of the error:
```
Failure/Error: association.target = records
```

and the same code in the `load_many` method:
```ruby
#(...)
association.target = records
#(...)
```

What's the difference? What's causing the:
```
undefined method `new_record?' for []:Array
```

It looks like the error is indeed something in batch-loader gem. Now the thing is to be able to identify the error in the stack trace.

## Fixed 🎉

Finally, fixed this by setting a guard clause in the `method_missing` in batch-loader. See [[007 - ✅ Exploring ActiveRecord Associations for AppBlend]]

# Only UUIDs are valid namespace identifiers ✅

This is for AWS SNS tests.

```bash
Failure/Error: Digest::UUID.uuid_v5(ENV.fetch("AWS_SNS_APP_#{platform.upcase}_#{app_type.upcase}"), device_token)

     ArgumentError:
       Only UUIDs are valid namespace identifiers
     Shared Example Group: "a Providers::AWS::SNS::PlatformEndpoints" called from ./spec/lib/notifications/providers/aws/sns/platform_endpoints/ios_therapist_spec.rb:4
     # ./spec/support/aws_sns_stubbed_methods.rb:136:in `mock_arn_token'
     # ./spec/support/aws_sns_stubbed_methods.rb:42:in `block (2 levels) in <module:StubbedMethods>'
     # ./spec/support/shared_examples/provider_aws_sns_platform_endpoint_shared_examples.rb:192:in `block (5 levels) in <top (required)>'
```

The mock is this:
```ruby
def mock_arn_token(device_token)
  Digest::UUID.uuid_v5(ENV.fetch("AWS_SNS_APP_#{platform.upcase}_#{app_type.upcase}"), device_token)
end
```

Is this about the `Digest::UUID` library? Looks to be an error with this library
```ruby
namespace = "arn:aws:sns:fake-region:1234657890:app/APNS/test-ios-therapist"
Digest::UUID.uuid_v5(namespace, "aaaaaaa")

ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/core_ext/digest/uuid.rb:64:in `pack_uuid_namespace':

Only UUIDs are valid namespace identifiers (ArgumentError)
```

Found this is a new config for Rails 7. See in the docs -> https://guides.rubyonrails.org/v7.0/configuring.html#config-active-support-use-rfc4122-namespaced-uuids

Found the clue via these issue reports:

- [Namespaced UUIDs are not generated correctly when the namespace ID provided is the string representation of a UUID](https://github.com/rails/rails/issues/37681)
- [Rails New Framework Default Documentation is Inadequate](https://github.com/rails/rails/issues/50238#issuecomment-1841562696)

Change the setting to false in the file `config/initializers/new_framework_defaults_7_0.rb`:
```ruby
Rails.application.config.active_support.use_rfc4122_namespaced_uuids = false
```

# Rails 7.0 ignores default format for Date and Time ✅

A setting like:
```ruby
# config/initializers/date_time.rb

Date::DATE_FORMATS[:default] = '%d/%m/%Y'
```

Working in Rails 6 would be ignored in Rails 7.0. As per the [release notes](https://github.com/rails/rails/blob/de7c495209811f8f918fa9f915e64df61a6080c1/guides/source/7_0_release_notes.md#deprecations-8):

> Deprecate passing a format to `to_s` in favor of `to_fs` in Array, Range, Date, DateTime, Time, BigDecimal, Float and, Integer.
>
> This deprecation is to allow Rails application to take advantage of a Ruby 3.1 optimization that makes interpolation of some types of objects faster.
>
> New applications will not have the `to_s` method overridden on those classes, existing applications can use `config.active_support.disable_to_s_conversion`.

The ideal fix is to find all places where dates are interpolated and send the `to_fs(:default)` message like this:
```ruby
"must be on or after #{latest_effective_until.to_fs(:default)},"
```

In [Stack Overflow](https://stackoverflow.com/questions/71177165/rails-ignores-the-default-date-format-after-upgrading-from-6-1-to-7-0).

## Disabling this does not bring back the functionality 🟡

> [!info]
> It's a bug fixed in Rails 7.0.7. See [[004 - Config Defaults#What is config.active_support.disable_to_s_conversion?]]

Even when disabled, this does not work as in Rails 6. As mentioned in [this answer](https://stackoverflow.com/a/71328079/1407371).

The solution is to find all places to replace `to_s` with `to_fs` or monkey patch Date class.

# Zeitwerk

## About Autoload and Reloadable Code

As [explained here](https://guides.rubyonrails.org/v7.0/autoloading_and_reloading_constants.html#autoloading-when-the-application-boots) and mentioned [here](https://stackoverflow.com/a/73463696/1407371) by Xavier Noira.

The Autoloading and Reloading Constants guides say:
> While booting, applications can autoload from the autoload once paths (...)
>
> However, ==you cannot autoload from the autoload paths== (...). This applies to code in `config/initializers` as well as application or engines initializers.

It also explains the why:
> Why? **Initializers only run once, when the application boots**. If you reboot the server, they run again in a new process, but ==reloading does not reboot the server, and initializers don't run again==.

Code in the `config/initializers` only loads when the Rails app boots.

To autoload code in this folder when the app boots and reload, you need to use the `to_prepare` block:
```ruby
# config/initializers/api_gateway_setup.rb
Rails.application.config.to_prepare do
  ApiGateway.endpoint = "https://example.com" # CORRECT
end
```

This callback might run twice.

To make code in the initializers folder load only on boot, use this other callback instead:
```ruby
Rails.application.config.after_initialize do
  ActionMailer::Base.register_observer(EmailLogHubspot)
end
```

The guides make it clear that:
> Reloadable classes and modules can be autoloaded in `after_initialize` blocks too. These run on boot, *but do not run again on reload*.

## Commands to check everything is in order

Tags: #zeitwerk_commands

```bash
bundle exec rails runner 'p Rails.autoloaders.zeitwerk_enabled?'
bundle exec rails zeitwerk:check
```

## Wrap initializers in callback ✅

I had to wrap lots of code in initializers folder with this block:
```ruby
Rails.application.config.after_initialize do
end
```

Because Zeitwerk changed the way code is loaded and as [Xavier Noira said](https://stackoverflow.com/a/73463720/1407371):
> This is unrelated to Zeitwerk, ==autoloading from initializers was just wrong conceptually regardless of the autoloader==.

## uninitialized constant EmailLogHubspot (zeitwerk) ✅

This code in `config/initializers/action_mailer`:
```ruby
ActionMailer::Base.register_observer(EmailLogHubspot)
```

causes this error:
```bash
NameError:
  uninitialized constant EmailLogHubspot

  ActionMailer::Base.register_observer(EmailLogHubspot)
                                       ^^^^^^^^^^^^^^^
# ./config/initializers/action_mailer.rb:3:in `<top (required)>'
# /Users/francisco/.gem/ruby/3.1.0/gems/bootsnap-1.10.3/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:48:in `load'
# /Users/francisco/.gem/ruby/3.1.0/gems/bootsnap-1.10.3/lib/bootsnap/load_path_cache/core_ext/kernel_require.rb:48:in `load'
```

when doing the Zeitwerk check command
```
bundle exec rails zeitwerk:check
```

This is fixed it by wrapping the initializer in this custom Rails code:
```ruby
Rails.application.config.after_initialize do
  ActionMailer::Base.register_observer(EmailLogHubspot)
end
```


# Happened but can be ignored

## 🚫 Database configuration Errors 🚫

Got this when running specs:
```bash
ActiveRecord::ConnectionNotEstablished:
       No connection pool for 'ActiveRecord::Base' found.
     # /Users/francisco/.gem/ruby/3.1.0/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
     # /Users/francisco/.gem/ruby/3.1.0/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
     # /Users/francisco/.gem/ruby/3.1.0/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
     # /Users/francisco/.gem/ruby/3.1.0/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
     # /Users/francisco/.gem/ruby/3.1.0/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
     # /Users/francisco/.gem/ruby/3.1.0/gems/webmock-3.18.1/lib/webmock/rspec.rb:37:in `block (2 levels) in <top (required)>'
```

It traces to the gems webmock and rspec-retry but it seems they're innocent.

Finding a solution was difficult but what I tried and let me move on was adding this line to `spec/rails_helper.rb`
```ruby
ActiveRecord::Base.establish_connection
```

Added it here:
```ruby
# Checks for pending migrations and applies them before tests are run.
# If you are not using ActiveRecord, you can remove this line.
begin
  ActiveRecord::Migration.maintain_test_schema!
  ActiveRecord::Base.establish_connection
rescue ActiveRecord::PendingMigrationError => e
  puts e.to_s.strip
  exit 1
end
```

but now I get
```bash
Failure/Error: ActiveRecord::Base.establish_connection

ActiveRecord::AdapterNotSpecified:
  The `test` database is not configured for the `test` environment.

    Available database configurations are:


# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/database_configurations.rb:177:in `resolve_symbol_connection'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/database_configurations.rb:127:in `resolve'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/connection_handling.rb:353:in `resolve_config_for_connection'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/connection_handling.rb:51:in `establish_connection'
# /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-import-1.3.0/lib/activerecord-import/import.rb:250:in `establish_connection'
# ./spec/rails_helper.rb:50:in `<top (required)>'
```

Another error that I think might be related is this one I get after running `rails db:test:prepare`:
```bash
rails aborted!
NoMethodError: undefined method `flat_map' for nil:NilClass

        db_configs = configs.flat_map do |env_name, config|
                            ^^^^^^^^^

Tasks: TOP => db:test:prepare => db:load_config
```

Get the same connection pool in dev rails console:
```bash
pry(main)> ActiveRecord::Base.connection.tables
ActiveRecord::ConnectionNotEstablished: No connection pool for 'ActiveRecord::Base' found.
```

**Update: unresolved -> changed to a different branch**

I wasn't able to fix this or find a solution that would help me move forward. I asked team mates for help and one of them used my branch, run the `rails app:update` command and somehow got a working version. I've continued my work using that branch. There's still errors. Looks like in my branch I was digging deeper into a rabbit hole.