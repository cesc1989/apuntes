# Upgrade to Rails 7.0.4 from 6.1.7 - Notes

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

