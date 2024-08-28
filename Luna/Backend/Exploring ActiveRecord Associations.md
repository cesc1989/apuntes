# Exploring ActiveRecord Associations for AppBlend

This is the output of exploring
```ruby
def load_many(records, association_name)
	association = self.association(association_name)
	association.loaded!
	association.target = records
	records.each { |record| association.set_inverse_instance(record) }
	self
end
```

> Note: this methods I can find them in `activerecord/lib/active_record/associations/association.rb`.
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

