# Browsing Batch Loader with debugger steps

Trying to understand [[Upgrade to Rails 7.0.4 - Notes Part 2 ✅#Error with PayerAuthorization and BatchLoader]]

I decided to debug from batch-loader gem:
```ruby
def method_missing(method_name, *args, **opts, &block)
    puts "Veamos que es sync: #{__sync!.class.name}"
    puts "Veamos que es method_name: #{method_name}"
    # return unless __sync!.respond_to?(method_name)
debugger
    __sync!.public_send(method_name, *args, **opts, &block)
  end
```

Without the guard clause, in Rails 7, it breaks right away and does not let me do puts debugging.

## Debugging in Rails 7.0.4

Once in the byebug session, I write `up` and get to `app/models/concerns/lazy_loadable.rb`.

But before, let's try to see what's the deal with the `new_record?` being sent to an Array instance. Why does it happen?

```bash
[72, 81] in /Users/francisco/.gem/ruby/3.1.0/bundler/gems/batch-loader-4a502c3f35fa/lib/batch_loader.rb
   72:   def method_missing(method_name, *args, **opts, &block)
   73:     puts "Veamos que es sync: #{__sync!.class.name}"
   74:     puts "Veamos que es method_name: #{method_name}"
   75:     # return unless __sync!.respond_to?(method_name)
   76: debugger
=> 77:     __sync!.public_send(method_name, *args, **opts, &block)
   78:   end
   79:
   80:   def __sync!
   81:     loaded_value = __sync
(byebug) __sync!.class.name
"Array"
(byebug) method_name
:[]
(byebug) c
Veamos que es sync: Episode
Veamos que es method_name: load_many

[72, 81] in /Users/francisco/.gem/ruby/3.1.0/bundler/gems/batch-loader-4a502c3f35fa/lib/batch_loader.rb
   72:   def method_missing(method_name, *args, **opts, &block)
   73:     puts "Veamos que es sync: #{__sync!.class.name}"
   74:     puts "Veamos que es method_name: #{method_name}"
   75:     # return unless __sync!.respond_to?(method_name)
   76: debugger
=> 77:     __sync!.public_send(method_name, *args, **opts, &block)
   78:   end
   79:
   80:   def __sync!
   81:     loaded_value = __sync
(byebug) __sync!.class.name
"Episode"
(byebug) method_name
:load_many
(byebug) c
Veamos que es sync: Array
Veamos que es method_name: new_record?

[72, 81] in /Users/francisco/.gem/ruby/3.1.0/bundler/gems/batch-loader-4a502c3f35fa/lib/batch_loader.rb
   72:   def method_missing(method_name, *args, **opts, &block)
   73:     puts "Veamos que es sync: #{__sync!.class.name}"
   74:     puts "Veamos que es method_name: #{method_name}"
   75:     # return unless __sync!.respond_to?(method_name)
   76: debugger
=> 77:     __sync!.public_send(method_name, *args, **opts, &block)
   78:   end
   79:
   80:   def __sync!
   81:     loaded_value = __sync
(byebug) __sync!.class.name
"Array"
(byebug) method_name
:new_record?
```

In this point, I do `up` once more and get to the place where the thing really breaks:
```bash
(byebug) up

[439, 448] in /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/associations/collection_association.rb
   439:
   440:           records
   441:         end
   442:
   443:         def replace_on_target(record, skip_callbacks, replace:, inversing: false)
=> 444:           if replace && (!record.new_record? || @replaced_or_added_targets.include?(record))
   445:             index = @target.index(record)
   446:           end
   447:
   448:           catch(:abort) do
(byebug) record
#<BatchLoader:0x727480>
(byebug) record.class
Veamos que es sync: Array
Veamos que es method_name: class
Array
```

The code snippet is:
```ruby
if replace && (!record.new_record? || @replaced_or_added_targets.include?(record))
```

And this is where it _actually_ fails:
```bash
(byebug) record.new_record?
Veamos que es sync: Array
Veamos que es method_name: new_record?
*** NoMethodError Exception: undefined method `new_record?' for [#<PayerAuthorization id: "a6909d94-d99f-425e-9c19-bbf259d63771", visit_amount: 2, authorization_number: "DQYWNOSHVZ", effective_until: "2019-04-14", reauthorization_visit_number: 1, required_gap_in_days: 5, request_status: "granted", episode_id: "1e832dff-4bcb-4216-9bd9-0d0d140b0ecc", created_at: "2018-10-14 11:00:00.000000000 +0000", updated_at: "2018-10-14 11:00:00.000000000 +0000", deleted_at: nil, submitted_at: "2018-10-14">]:Array

    __sync!.public_send(method_name, *args, **opts, &block)
           ^^^^^^^^^^^^

nil
```

Why does it not fail in Rails 6.1.6?

I kept going `up`:
```bash
(byebug) up

[279, 288] in /Users/francisco/.gem/ruby/3.1.0/gems/activerecord-7.0.4/lib/active_record/associations/collection_association.rb
   279:         when nil
   280:           # It's not possible to remove the record from the inverse association.
   281:         when Array
   282:           super
   283:         else
=> 284:           replace_on_target(record, true, replace: true, inversing: true)
   285:         end
   286:       end
   287:
   288:       def scope
(byebug) record
#<BatchLoader:0x727480>
(byebug) record.class
Veamos que es sync: Array
Veamos que es method_name: class
Array
(byebug) up

[5, 14] in /Users/francisco/projects/luna-project/backend/app/lib/active_record_extensions/query_methods/with_load_methods.rb
    5:     # Allows associations to be loaded into this object retroactively.
    6:     module WithLoadMethods
    7:       def load_many(records, association_name)
    8:         association = self.association(association_name)
    9:         association.loaded!
=> 10:         association.target = records
   11:         records.each { |record| association.set_inverse_instance(record) }
   12:         self
   13:       end
   14:
(byebug) records
#<BatchLoader:0x727480>
```

Surprisingly, in `activerecord-7.0.4/lib/active_record/associations/collection_association.rb` line 278 there's this [code](https://github.com/rails/rails/blob/v7.0.4/activerecord/lib/active_record/associations/collection_association.rb#L278):
```ruby
case record
when nil
	# It's not possible to remove the record from the inverse association.
when Array
	super
else
	replace_on_target(record, true, replace: true, inversing: true)
end
```

and for some reason it does `replace_on_target`, the else case, instead of the `Array` case. wtf?

Finally, when we're back in the ActiveRecordExtensions module we can see that `records` is an array that's being assigned to `targets=`. Which is the method containing the switch case described above.

Why is the switch case calling `replace_on_target`?

## Let's see this in Rails 6.1.6

Here's what looks to be the big difference between the two versions:
```bash
Usado en Rails 6.1.6
Veamos que es sync: Array
Veamos que es method_name: []

[73, 82] in /Users/francisco/.gem/ruby/3.1.0/bundler/gems/batch-loader-350767424460/lib/batch_loader.rb
   73:     puts "Usado en Rails 6.1.6"
   74:     puts "Veamos que es sync: #{__sync!.class.name}"
   75:     puts "Veamos que es method_name: #{method_name}"
   76:     # return unless __sync!.respond_to?(method_name) # copia
   77: debugger
=> 78:     __sync!.public_send(method_name, *args, **opts, &block)
   79:   end
   80:
   81:   def __sync!
   82:     loaded_value = __sync
(byebug) __sync!.class.name
"Array"
(byebug) method_name
:[]
(byebug) c
Veamos que es sync: Episode
Veamos que es method_name: load_many

[73, 82] in /Users/francisco/.gem/ruby/3.1.0/bundler/gems/batch-loader-350767424460/lib/batch_loader.rb
   73:     puts "Usado en Rails 6.1.6"
   74:     puts "Veamos que es sync: #{__sync!.class.name}"
   75:     puts "Veamos que es method_name: #{method_name}"
   76:     # return unless __sync!.respond_to?(method_name) # copia
   77: debugger
=> 78:     __sync!.public_send(method_name, *args, **opts, &block)
   79:   end
   80:
   81:   def __sync!
   82:     loaded_value = __sync
(byebug) __sync!.class.name
"Episode"
(byebug) method_name
:load_many
(byebug) c
Usado en Rails 6.1.6
Veamos que es sync: Array
Veamos que es method_name: each
```

In the third inspection:
```
Veamos que es sync: Array
Veamos que es method_name: each
```

Instead of sending `new_record?` to Array, it's sending `each`. Why???

When I go `up` it goes straight to ActiveRecordExtensions module:
```bash
(byebug) up

[6, 15] in /Users/francisco/projects/luna-project/omeguitis/app/lib/active_record_extensions/query_methods/with_load_methods.rb
    6:     module WithLoadMethods
    7:       def load_many(records, association_name)
    8:         association = self.association(association_name)
    9:         association.loaded!
   10:         association.target = records
=> 11:         records.each { |record| association.set_inverse_instance(record) }
   12:         self
   13:       end
   14:
   15:       def load_one(record, association_name)
```

It does nothing at `association.target = records` and continues to:
```ruby
records.each { |record| association.set_inverse_instance(record) }
```

Is it that in Rails 7.0.4, for some reason, it's ignoring the check:
```ruby
case record
when nil
	# It's not possible to remove the record from the inverse association.
when Array
	super
else
	replace_on_target(record, true, replace: true, inversing: true)
end
```

and it does it correctly in Rails 6.1.6?

## has_many_inversing

In Rails 6.1.6, in the `target=` method:
```ruby
def target=(record)
  return super unless ActiveRecord::Base.has_many_inversing
end
```

> See [source](https://github.com/rails/rails/blob/v6.1.6/activerecord/lib/active_record/associations/collection_association.rb#L286-L287)

returns false:
```bash
(byebug) ActiveRecord::Base.has_many_inversing
false
```

However, in 7.0.4:
```ruby
def target=(record)
	return super unless reflection.klass.has_many_inversing
end
```

> See [source](https://github.com/rails/rails/blob/v7.0.4/activerecord/lib/active_record/associations/collection_association.rb#L275-L276)

returns true:
```bash
(byebug) reflection.klass.has_many_inversing
true
```

This is why it is not going into the switch case.

## A workaround // fix

Found something related in this issue report. In one of the comments they mentioned this config:
```ruby
config.active_record.has_many_inversing = false
```

When adding that to `config/application.rb` it stops the errors.

Now I have to send this to the CI and see what happens.

Related links:

- Eager loading bug with inverse_of and the same model in the query twice [50258](https://github.com/rails/rails/issues/50258)
- Attempting to save duplicate records with has_many_inversing enabled leads to SQL error [47559](https://github.com/rails/rails/issues/47559)
- Comment on Porta project. [Code review comment](https://github.com/3scale/porta/pull/3705#pullrequestreview-2003734742)

### Comment on Porta project

Bringing the text here just in case:

mayorova:
> The issues I found are commented in the places they occur, search for has_many_inversing.
>
> Also, it seems that there are some open issues related to this setting:
> rails/rails#50258
> rails/rails#47559
> 
> and maybe more. So I think the safest way would be to keep it permanently, until a deeper review of this is performed.

akostadinov:
> So without `has_many_inversing` what is the behavior? Yeah, it seems we better keep this forever. No need to change it. We can set `has_one` or `has_many` whenever needed but doing it automatically appears to be harmful as we can't override at some place but keep in other places.
>
> I would suggest adding a comment about it.