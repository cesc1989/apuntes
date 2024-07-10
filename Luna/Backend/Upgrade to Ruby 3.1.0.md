# Upgrade to Ruby 3.1.0

# Errors Found and Solutions

## Missing net/smtp

This error:
```
You don't have net-smtp installed in your application. Please add it to your Gemfile and run bundle install
rails aborted!
LoadError: cannot load such file -- net/smtp
```

Solution is to add it to the Gemfile
```ruby
gem 'net-smtp', require: false
```

Note: depending on other gems and what libraries are used, you might need to also install:
- net-imap
- net-pop
- net-sftp

These libraries are being excluded from Ruby core so that it's easier for the maintainers team to make changes to Ruby like security reports.

From [reddit](https://www.reddit.com/r/ruby/comments/1d7xn3q/comment/l72dpa3/):
> Main reason is that it is easier to maintain and update the csv library as a gem when it is not part of the standard library. Ruby is released once a year with significant updates and a handful of patch releases. Removing the gems from being bundled with ruby, allows them to be updated/patched seperately and also does not warrant a ruby update when a major security problem is found in the gem.
>
> They already did that over the last year with things like the `net-*` libraries (ssh, smtp, etc)

## Timecop error

Got this error about Timecop:
```
rails aborted!
TypeError: no implicit conversion of Hash into Integer
<internal:timev>:310:in `initialize'
/usr/local/bundle/gems/timecop-0.9.2/lib/timecop/time_extensions.rb:22:in `new'
/usr/local/bundle/gems/timecop-0.9.2/lib/timecop/time_extensions.rb:22:in `new_with_mock_time'
<internal:timev>:224:in `now'
/usr/local/bundle/gems/timecop-0.9.2/lib/timecop/time_extensions.rb:14:in `now_with_mock_time'
```

Found [this issue](https://github.com/travisjeffery/timecop/issues/372) describing same error. Looks like it was fixed in version 0.9.4.

There were [some PRs](https://github.com/travisjeffery/timecop/pull/279) about it but it is not clearly stated that version fixes it but it does after trying in the CI.

## Rubocop error

This one:
```
Error: RuboCop found unknown Ruby version 3.1 in `.ruby-version`.

[137](https://github.com/lunacare/backend/actions/runs/9875933856/job/27273893482#step:4:138)Supported versions: 2.4, 2.5, 2.6, 2.7, 3.0

[138](https://github.com/lunacare/backend/actions/runs/9875933856/job/27273893482#step:4:139)RuboCop exited unsuccessfully with status 2. Check the stack trace to see if there was a misconfiguration.
```

This can be fixed by upgrading Rubocop to version 1.14.0. This is the minimum version where Ruby 3.1.0 is supported. See [code](https://github.com/rubocop/rubocop/blob/v1.14.0/lib/rubocop/target_ruby.rb#L7).
