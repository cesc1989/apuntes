# Upgrade to Rails 7.0.4 - Notes

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