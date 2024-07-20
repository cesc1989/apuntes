# Upgrade to Ruby 3.1.0

# Errors and Solutions

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

## Missing net/pop

Got this error in Argocd instead of the CI:
```
core_ext/kernel_require.rb:27:in `require': cannot load such file -- net/pop (LoadError)
```

As mentioned before, looks like all these gems are required by the Mail gem.

Solution? Add gem net-pop:
```ruby
gem "net-pop", require: false
```

## Missing net/imap

Got this error in Argocd instead of the CI:
```
core_ext/kernel_require.rb:27:in `require': cannot load such file -- net/imap (LoadError)
```

As mentioned before, looks like all these gems are required by the Mail gem.

Solution? Add gem net-imap:
```ruby
gem "net-imap", require: false
```

## What's the deal with the net-* gems?

In [this Stack Overflow](https://stackoverflow.com/a/70500221/1407371) answer the OP mentions that upgrading the mail gem to version 2.8.0 would fix it. Look at this project's version:

```
$ cat Gemfile.lock | grep mail
    actionmailbox (6.1.6)
      mail (>= 2.7.1)
    actionmailer (6.1.6)
      mail (~> 2.5, >= 2.5.4)
    mail (2.7.1)
```

It's not using the stated version and it's why the error happens for all net-* gems. So there are two options: add the gems individually or upgrade the mail gem (conservatively).

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

## Rubocop errors

This one:
```
Error: RuboCop found unknown Ruby version 3.1 in `.ruby-version`.

[137](https://github.com/lunacare/backend/actions/runs/9875933856/job/27273893482#step:4:138)Supported versions: 2.4, 2.5, 2.6, 2.7, 3.0

[138](https://github.com/lunacare/backend/actions/runs/9875933856/job/27273893482#step:4:139)RuboCop exited unsuccessfully with status 2. Check the stack trace to see if there was a misconfiguration.
```

This can be fixed by upgrading Rubocop to version 1.14.0. This is the minimum version where Ruby 3.1.0 is supported. See [code](https://github.com/rubocop/rubocop/blob/v1.14.0/lib/rubocop/target_ruby.rb#L7).

### An error occurred while Layout/BlockAlignment cop was inspecting

This appeared for all of the files in the project:
```
An error occurred while Layout/BlockAlignment cop was inspecting /application/spec/workers/worker_inheritance_spec.rb:6:0.
An error occurred while Layout/BlockAlignment cop was inspecting /application/spec/workers/workout_notification_worker_spec.rb:3:0.
An error occurred while Layout/BlockAlignment cop was inspecting /application/spec/workers/workouts_reminder_worker_spec.rb:3:0.
Errors are usually caused by RuboCop bugs.
Please, report your problems to RuboCop's issue tracker.
https://github.com/rubocop/rubocop/issues

Mention the following information in the issue report:
1.14.0 (using Parser 3.3.3.0, rubocop-ast 1.31.3, running on ruby 3.1.0 x86_64-linux-musl)
```

but only seems to happen in Linux and the cops still passed.
```
4127 files inspected, no offenses detected
```

According to [this comment](https://github.com/rubocop/rubocop/issues/10599#issuecomment-1116276917) it's fixed on version 1.28.2

And it does :D

## Shoulda-matchers at master@3e9d69b is not yet checked out

This is only happens in Argo. This is the error:
```
/usr/local/bundle/gems/bundler-2.4.22/lib/bundler/source/git.rb:221:in `rescue in load_spec_files': https://github.com/thoughtbot/shoulda-matchers (at master@3e9d69b) is not yet checked out. Run `bundle install` first. (Bundler::GitError)
```

When looking at the Gemfile I see shoulda-matchers is installed this way:
```ruby
gem "shoulda-matchers", git: "https://github.com/thoughtbot/shoulda-matchers", branch: "master"
```

However, in the repo the "master" branch is "main" and (as of today, 20th July, 2024) it is:
> This branch is 110 commits behind main.

There's two options here. Figure out the version master was left at or change the branch to main.

In the file `lib/shoulda/matchers/version.rb` I could [see](https://github.com/thoughtbot/shoulda-matchers/blob/master/lib/shoulda/matchers/version.rb) the version to be 5.0.0
```ruby
module Shoulda
  module Matchers
    # @private
    VERSION = '5.0.0'.freeze
  end
end
```

This can also be verified by looking at the Gemfile.lock file:
```
GIT
  remote: https://github.com/thoughtbot/shoulda-matchers
  revision: 3e9d69ba7b6cc37251deb08f1a8866c97cf2bc0c
  branch: master
  specs:
    shoulda-matchers (5.0.0)
      activesupport (>= 5.2.0)
```

So, let's change it to point to version instead.

> As a note, I checked the git history and last time a commit affected the should-matchers installation was at 2019-11-21 and prior to that time it was still pointing to master branch.
>
> Looks like at the time it was ok to have it pointing to master but then it happen that people wanted "master" to be renamed to other thing because of the implication of a master-slave.