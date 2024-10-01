# Exploring ActiveRecord Associations for AppBlend

> [!INFO]
> The correct fix for this is in [[Browsing Batch Loader in Rails 7.0.4 ✅]]
>
> What's in this document is good but not the final fix.

This is the output of exploring `ActiveRecordExtensions` module.
```ruby
def load_many(records, association_name)
	association = self.association(association_name)
	association.loaded!
	association.target = records
	records.each { |record| association.set_inverse_instance(record) }
	self
end
```

> Note: these methods I can find them in `activerecord/lib/active_record/associations/association.rb`.
> 
> In this file I find methods: `loaded!`, `target=`, and `set_inverse_instance(record)`.

## Outputs

For this test:
```
pruebas ./spec/requests/graphql/mutations/scheduling/bulk_add_appointment_spec.rb:106
```

What is `records`?
```ruby
ap records
[
    #<Patient:0x000000013cdb5cd0> {
        :id => "19be6b36-f7fa-4e5b-8cc4-36fc115b5a18",
    }
]
```

What is `association_name`?
```ruby
ap association_name
:patients
```

What is
```ruby
association = self.association(association_name)
```

What is `self` in this context?
```ruby
ap self
#<Therapist:0x000000013cd0f498> {
	"id" => "55f09374-0547-4608-b23c-e4499fee2dcf",
}
```

What is `association`?
```ruby
ap association

<ActiveRecord::Associations::HasManyAssociation:0x000000013cd0e818
  @reflection=<ActiveRecord::Reflection::HasManyReflection:0x000000013baaeee8
    @name=:patients,
    @scope=<Proc:0x000000013baaf168 /Users/francisco/projects/luna-project/backend/app/models/therapist.rb:117 (lambda)>,
    @options={:class_name=>"Patient"},
    @active_record=Therapist(),
    @klass=Patient(),
    @plural_name="patients",
    @class_name="Patient",
    @foreign_key="therapist_id",
    @inverse_name=nil,
    @active_record_primary_key="id"
  >,
  @owner=<Therapist id: "55f09374-0547-4608-b23c-e4499fee2dcf">,
  @disable_joins=false,
  @loaded=true,
  @target=[<Patient id: "19be6b36-f7fa-4e5b-8cc4-36fc115b5a18">],
  @stale_state=nil,
  @replaced_or_added_targets=<Set: {}>,
  @association_ids=nil,
  @association_scope=<ActiveRecord::Relation [<>]>,
  @proxy=<ActiveRecord::Associations::CollectionProxy [<Patient id: "19be6b36-f7fa-4e5b-8cc4-36fc115b5a18"]>
>
```

What is `association.target`?
```ruby
ap association.target
[
    <Patient:0x000000013cdb5cd0> {
      :id => "19be6b36-f7fa-4e5b-8cc4-36fc115b5a18",
    }
]
```

## Outputs in batch-loader steps

In the test stack trace, it steps in the private `method_missing`:
```bash
Failure/Error: association.target = records

     NoMethodError:
       undefined method new_record? for []:Array

           __sync!.public_send(method_name, *args, **opts, &block)
                  ^^^^^^^^^^^^
     # /Users/francisco/.gem/ruby/3.1.0/bundler/gems/batch-loader-350767424460/lib/batch_loader.rb:77:in public_send
     # /Users/francisco/.gem/ruby/3.1.0/bundler/gems/batch-loader-350767424460/lib/batch_loader.rb:77:in method_missing
     # ./app/lib/active_record_extensions/query_methods/with_load_methods.rb:10:in `load_many'
```

It does so after passing by the `load_many` method in `ActiveRecordExtensions` module.

When inspecting in detail the part that breaks I can see this:
```bash
[70, 79] in /Users/francisco/.gem/ruby/3.1.0/bundler/gems/batch-loader-350767424460/lib/batch_loader.rb
   70:   end
   71:
   72:   def method_missing(method_name, *args, **opts, &block)
   73:     debugger if method_name == :new_record?
   74:
=> 75:     __sync!.public_send(method_name, *args, **opts, &block)
   76:   end
   77:
   78:   def __sync!
   79:     loaded_value = __sync
(byebug) __sync!
[]
(byebug) method_name
:new_record?
(byebug) args
[]
(byebug) opts
{}
(byebug) block
nil
(byebug)
```

Why is `__sync!` an empty array in batch-loader? In previous steps I could see multiple instances of ActiveRecord:
```bash
(byebug) __sync!
#<Episode id: "299a13c7-306e-44b9-ab48-5099e91bf209">

(byebug) __sync!
#<Therapist id: "fd4a5842-71cc-4cad-93c3-48f99ab94348">

(byebug) __sync!
#<Region id: 11, name: "new_york">
```

The test is failing because, in some moment it's trying to do:
```ruby
(byebug) __sync!.public_send(method_name)
*** NoMethodError Exception: undefined method `new_record?' for []:Array

nil
```

which obviously fails because Ruby `Array` does not implements `new_record?` instance method.

## Fixed

Added a guard clause in the `method_missing` definition:

```diff
   def method_missing(method_name, *args, **opts, &block)
+    return unless __sync!.respond_to?(method_name)
+
     __sync!.public_send(method_name, *args, **opts, &block)
   end
```