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

## The Fix 🩹

This is fixed in Rails 7.1.3. See [Action Pack](https://github.com/rails/rails/releases/tag/v7.1.3):
```
- Fix including `Rails.application.routes.url_helpers` directly in an  
    `ActiveSupport::Concern.`
    
    Jonathan Hefner
```

# 🎉 You tried to define an enum named "status" on the model but this will generate a instance method "frozen?", which is already defined by Active Record 🎉

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

## The Fix 🩹

To fix this, either change the word `frozen` in the enum or [add a prefix/suffix](https://api.rubyonrails.org/v7.1.4/classes/ActiveRecord/Enum.html).

# Your bundle is locked to mutex_m (0.3.0) from rubygems repository

```
Your bundle is locked to mutex_m (0.3.0) from rubygems repository
https://rubygems.org/ or installed locally, but that version can no longer be
found in that source. That means the author of mutex_m (0.3.0) has removed it.
You'll need to update your bundle to a version other than mutex_m (0.3.0) that
hasn't been removed in order to install.
```

This error went away without me doing anything.

# ArgumentError for ActiveRecord-Import gem

This error in tests:

```bash
 1) TherapistBadges achieved badges should return only the correct achieved badge of each type
     Failure/Error:
       NotificationSetting.kinds.each_key
         .map { |kind| { account_id: id, kind: kind, communication_methods: ["push"], enabled: true } }
         .then { |settings| NotificationSetting.import(settings, on_duplicate_key_ignore: true) }

     ArgumentError:
       wrong number of arguments (given 0, expected 1)
     # ./app/models/therapist.rb:1223:in `block in set_notification_setting_defaults'
     # ./app/models/therapist.rb:1223:in `set_notification_setting_defaults'
```

## The Fix 🩹

Fixed by upgrading activerecord-import gem to 1.5.0.

# undefined method table_name for ActiveRecord::SchemaMigration:Class

This:

```bash
Failure/Error: DatabaseCleaner.clean_with(:truncation)

     NoMethodError:
       undefined method `table_name' for ActiveRecord::SchemaMigration:Class

               ::ActiveRecord::SchemaMigration.table_name
                                              ^^^^^^^^^^^
     # /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-active_record-2.0.1/lib/database_cleaner/active_record/base.rb:18:in `migration_table_name'
     # /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-active_record-2.0.1/lib/database_cleaner/active_record/base.rb:23:in `exclusion_condition'
     # /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-active_record-2.0.1/lib/database_cleaner/active_record/truncation.rb:239:in `tables_with_schema'
```

The first search results pointed to Ransack but ActiveAdmin was already upgraded to version 3.1.0 and so Ransack is upgraded as well to a version that fixes it.

Ransack repo links:
- Issue: https://github.com/activerecord-hackery/ransack/issues/1444
- Release of 4.2.1 https://github.com/activerecord-hackery/ransack/releases/tag/v4.2.1

Gemfile status
```
# Gemfile.lock
ransack (4.2.1)
      activerecord (>= 6.1.5)
      activesupport (>= 6.1.5)
      i18n

# Gemfile
gem "activeadmin", "3.1.0"
```

So I read more the log and it looks like to be database-cleaner.

## The Fix 🩹

The fix is to upgrade gem to database_cleaner-active_record to 2.1.0 as [mentioned](https://github.com/DatabaseCleaner/database_cleaner-active_record/issues/83#issuecomment-1464759691).

# ArgumentError: Only UUIDs are valid namespace identifiers

```bash
Failure/Error: Digest::UUID.uuid_v5(ENV.fetch("AWS_SNS_APP_#{platform.upcase}_#{app_type.upcase}"), device_token)

      ArgumentError:
        Only UUIDs are valid namespace identifiers
      # ./spec/support/aws_sns_stubbed_methods.rb:136:in `mock_arn_token'
      # ./spec/support/aws_sns_stubbed_methods.rb:22:in `block (2 levels) in <module:StubbedMethods>'
```

This cannot be disabled with settings because the deprecated support is removed from Rails 7.1.4.

See
- [[01 - Notable changes from 7.1.0 to Rails 7.2#7.1.0]]
- [[004 - 👌🏽 AppBlend What Changes in Config Defaults#What is config.active_support.use_rfc4122_namespaced_uuids?]]

Need to change it to a compliant UUID.

## Context and Fix 🩹

The stub uses the method `Digest::UUID.uuid_v5`. What does it do?

Docs -> https://api.rubyonrails.org/classes/Digest/UUID.html#method-c-uuid_v5

> [!Note]
> `uuid_v5(uuid_namespace, name)`
>
> Convenience method for uuid_from_hash using OpenSSL::Digest::SHA1.

In the [Rails issue](https://github.com/rails/rails/issues/37681) and [PR fix](https://github.com/rails/rails/pull/37682) I see OP doing this:
```ruby
Digest::UUID.uuid_v5(Digest::UUID::DNS_NAMESPACE, "www.widgets.com")
```

Besides, in [this other commit](https://github.com/mysociety/alaveteli/pull/6915/files) from some random project they pass a UUID as the namespace part
```ruby
module User::LoginToken
  extend ActiveSupport::Concern

  LOGIN_TOKEN_NAMESPACE = 'b14cba73-a392-4de4-a9ed-06d7f0ced429'

  def set_login_token!
    self.login_token = Digest::UUID.uuid_v5(
      LOGIN_TOKEN_NAMESPACE,
      {
        user: id,
        email: email,
        hashed_password: hashed_password
      }.to_s
    )
  end
end
```

So I tried something similar for the `uuid_namespace` parameter:
```ruby
Digest::UUID.uuid_v5(Digest::UUID::DNS_NAMESPACE, ENV.fetch("AWS_SNS_APP_#{platform.upcase}_#{app_type.upcase}"))
```

And it made tests pass. Is it correct and enough?

# undefined method show_exceptions? for ActionDispatch::Request POST "http://www.example.com/graphql for 127.0.0.1

This big ass error:
```bash
1) node { ... on Therapist } → Conversation returns the therapist and their conversation
Failure/Error: post endpoint, params: params, headers: headers

NoMethodError:
  undefined method `show_exceptions?' for #<ActionDispatch::Request POST "http://www.example.com/graphql" for 127.0.0.1>

          return if request.show_exceptions? && !Sentry.configuration.rails.report_rescued_exceptions
                           ^^^^^^^^^^^^^^^^^
# /usr/local/bundle/gems/sentry-rails-5.7.0/lib/sentry/rails/capture_exceptions.rb:27:in `capture_exception'
# /usr/local/bundle/gems/sentry-ruby-5.7.0/lib/sentry/rack/capture_exceptions.rb:33:in `rescue in block (2 levels) in call'
# /usr/local/bundle/gems/sentry-ruby-5.7.0/lib/sentry/rack/capture_exceptions.rb:27:in `block (2 levels) in call'
# /usr/local/bundle/gems/sentry-ruby-5.7.0/lib/sentry/hub.rb:220:in `with_session_tracking'
# /usr/local/bundle/gems/sentry-ruby-5.7.0/lib/sentry-ruby.rb:375:in `with_session_tracking'
# /usr/local/bundle/gems/sentry-ruby-5.7.0/lib/sentry/rack/capture_exceptions.rb:19:in `block in call'
# /usr/local/bundle/gems/sentry-ruby-5.7.0/lib/sentry/hub.rb:59:in `with_scope'
# /usr/local/bundle/gems/sentry-ruby-5.7.0/lib/sentry-ruby.rb:355:in `with_scope'
# /usr/local/bundle/gems/sentry-ruby-5.7.0/lib/sentry/rack/capture_exceptions.rb:18:in `call'
# /usr/local/bundle/gems/railties-7.1.4/lib/rails/rack/logger.rb:37:in `call_app'
# /usr/local/bundle/gems/railties-7.1.4/lib/rails/rack/logger.rb:24:in `block in call'
# /usr/local/bundle/gems/railties-7.1.4/lib/rails/rack/logger.rb:24:in `call'
# /usr/local/bundle/gems/request_store-1.7.0/lib/request_store/middleware.rb:19:in `call'
# /usr/local/bundle/gems/rack-2.2.10/lib/rack/method_override.rb:24:in `call'
# /usr/local/bundle/gems/rack-2.2.10/lib/rack/runtime.rb:22:in `call'
# /usr/local/bundle/gems/rack-2.2.10/lib/rack/sendfile.rb:110:in `call'
# /usr/local/bundle/gems/rack-cors-2.0.1/lib/rack/cors.rb:102:in `call'
# /usr/local/bundle/bundler/gems/secure_headers-7a23cb6b350b/lib/secure_headers/middleware.rb:11:in `call'
# /usr/local/bundle/gems/railties-7.1.4/lib/rails/engine.rb:536:in `call'
# /usr/local/bundle/gems/rack-test-2.1.0/lib/rack/test.rb:360:in `process_request'
# /usr/local/bundle/gems/rack-test-2.1.0/lib/rack/test.rb:153:in `request'
# /usr/local/bundle/gems/rails-controller-testing-1.0.5/lib/rails/controller/testing/integration.rb:16:in `block (2 levels) in <module:Integration>'
# ./spec/support/graphql_helpers.rb:5:in `load_data_strictly'
# ./spec/requests/graphql/queries/fields/node_therapist_query_spec.rb:62:in `block (3 levels) in <top (required)>'
# /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
# /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
# /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
# /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
# /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
# /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
# ------------------
# --- Caused by: ---
# PG::UndefinedColumn:
#   ERROR:  column patients.patient_id does not exist
#   LINE 1: ...CT DISTINCT "patients"."id" FROM "patients" WHERE "patients"...
#                                                                ^
#   /usr/local/bundle/gems/bullet-7.1.4/lib/bullet/active_record71.rb:46:in `records'
```

In this test:
```
pruebas ./spec/requests/graphql/queries/fields/node_therapist_query_spec.rb

pruebas ./spec/requests/api/v1/internal/provider_communications_spec.rb

pruebas ./spec/controllers/api/v1/therapist/careplans_controller_spec.rb

pruebas ./spec/requests/api/v2/external/phaxio_spec.rb

pruebas ./spec/requests/api/v1/internal/patient_communications_spec.rb
```

# undefined method hgetall for ConnectionPool

```bash
1) Metrics::Provider#read_metric when using redis rails cache when the value is present in the cache reads the metric value from the cache
Failure/Error: cache.redis.hgetall(definition.redis_key)

NoMethodError:
  undefined method `hgetall' for #<ConnectionPool:0x00007f34e932a6e8 @size=5, @timeout=5, @auto_reload_after_fork=true, @available=#<ConnectionPool::TimedStack:0x00007f34e932a4b8 @create_block=#<Proc:0x00007f34e932a648 /usr/local/bundle/gems/activesupport-7.1.4/lib/active_support/cache/redis_cache_store.rb:153>, @created=0, @que=[], @max=5, @mutex=#<Thread::Mutex:0x00007f34e932a008>, @resource=#<Thread::ConditionVariable:0x00007f34e9329f40>, @shutdown_block=nil>, @key=:"pool-3957680", @key_count=:"pool-3957680-count">

                cache.redis.hgetall(definition.redis_key)
                           ^^^^^^^^
# ./app/services/metrics/provider.rb:41:in `read_metric'
# ./spec/services/metrics/provider_spec.rb:119:in `block (5 levels) in <top (required)>'
```

In this test:
```
pruebas ./spec/services/metrics/provider_spec.rb
```

# PG::UndefinedColumn: ERROR: column patients.patient_id does not exist

```bash
1) Therapist scopes .conversable_patients does not exclude patients with care plans that are scheduling disabled
Failure/Error: expect(therapist.reload.conversable_patients).to eq [patient]

ActiveRecord::StatementInvalid:
 PG::UndefinedColumn: ERROR:  column patients.patient_id does not exist
 LINE 1: ...ELECT DISTINCT "patients".* FROM "patients" WHERE "patients"...
                                                              ^
# /usr/local/bundle/gems/bullet-7.1.4/lib/bullet/active_record71.rb:46:in `records'
# /usr/local/bundle/gems/bullet-7.1.4/lib/bullet/active_record71.rb:202:in `load_target'
# ./spec/models/therapist_spec.rb:42:in `block (4 levels) in <top (required)>'
# /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:124:in `block in run'
# /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `loop'
# /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:110:in `run'
# /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec_ext/rspec_ext.rb:12:in `run_with_retry'
# /usr/local/bundle/gems/rspec-retry-0.6.2/lib/rspec/retry.rb:37:in `block (2 levels) in setup'
# /usr/local/bundle/gems/webmock-3.23.1/lib/webmock/rspec.rb:39:in `block (2 levels) in <top (required)>'
# ------------------
# --- Caused by: ---
# PG::UndefinedColumn:
#   ERROR:  column patients.patient_id does not exist
#   LINE 1: ...ELECT DISTINCT "patients".* FROM "patients" WHERE "patients"...
#                                                                ^
#   /usr/local/bundle/gems/bullet-7.1.4/lib/bullet/active_record71.rb:46:in `records'
```

In this test:
```
pruebas ./spec/models/therapist_spec.rb

pruebas ./spec/controllers/api/v3/therapist/charts_controller_spec.rb

pruebas ./spec/workers/hidden_therapist_conversations_reading_worker_spec.rb

pruebas ./spec/requests/graphql/queries/fields/viewer_therapist_query_spec.rb

pruebas ./spec/controllers/api/v1/patient/conversations_controller_spec.rb

pruebas ./spec/requests/admin/scheduler_therapists_spec.rb
```

