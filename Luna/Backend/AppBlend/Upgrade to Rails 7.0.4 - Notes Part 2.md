# Upgrade to Rails 7.0.4 - Notes Part 2

This is a continuation of [[Upgrade to Rails 7.0.4 - Notes]]

# Test Errors

## Unsafe redirect to "https://www.getluna.com", pass allow_other_host: true to redirect anyway

Error:
```
 1) PaymentsController Actions redirects without an appointment id `aid` should redirect to getluna.com
     Failure/Error: redirect_to "https://www.getluna.com"

     ActionController::Redirecting::UnsafeRedirectError:
       Unsafe redirect to "https://www.getluna.com", pass allow_other_host: true to redirect anyway.
     # /Users/francisco/.gem/ruby/3.1.0/gems/turbolinks-5.2.1/lib/turbolinks/redirection.rb:12:in `redirect_to'
     # ./app/controllers/payments_controller.rb:9:in `index'
```

Fix:
```ruby
redirect_to("https://www.getluna.com", allow_other_host: true)
```

About: https://blog.saeloun.com/2022/02/08/rails-7-raise-unsafe-redirect-error/

## Value not defined un Enum raises ActiveRecord::NotNullViolation with null constraint

Ran this and got this error:
```bash
pruebas ./spec/serializers/admin/patient_serviceability_check_serializer_spec.rb

Failure/Error:
       check = create(
         :patient_serviceability_check,
         referral_source: :partner,
         referral_source_entity: practice
       )

     ActiveRecord::NotNullViolation:
       PG::NotNullViolation: ERROR:  null value in column "smart_routing_clinic_category" of relation "patient_serviceability_checks" violates not-null constraint
       DETAIL:  Failing row contains (b0b6b590-ca85-4f75-a3e9-62628ea3f222, 0, 0, null, Practice, 115eac23-367f-42af-bea8-25743325313a, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, null, 2018-10-14 11:00:00, 2018-10-14 11:00:00, null, null, {}, null, null, null, null, null, null, null, null, null, f, f).
```

When checking the schema of the model and the enum definition
```ruby
# schema
t.integer "smart_routing_clinic_category", default: 0, null: false

# model
enum smart_routing_clinic_category: {
  in_network: 1,
  out_of_network: 2,
  self_pay: 3
}
```

I found that there's no defined value in the enum when the record value is zero. Because of this and the `null: false` constraint in action the creation fails in Rails 7.

In [this issue](https://github.com/rails/rails/issues/52074) someone asked this same problem and the explanation is that Rails now expects enum values to match DB values.

## Error with PayerAuthorization and BatchLoader

Association: Episode -> PayerAuthorizations (aliased Authorizations).

To create Authorization there's this validation:
```ruby
validate :validate_reauthorization_visit_number

def validate_reauthorization_visit_number
  errors.add(:reauthorization_visit_number, "must be present") && return if reauthorization_visit_number.blank?


  min = episode.authorizations.not_denied.where.not(id: id).sum(&:visit_amount) + 1
  max = min + visit_amount - 1


  if min == max
    errors.add(:reauthorization_visit_number, "must be exactly #{min}")
  else
    errors.add(:reauthorization_visit_number, "must be between #{min} and #{max}")
  end
end
```

> Aside: In the tests about the `gap` variable I was unable to create authorization because I'd always get an error about `"must be between #{min} and #{max}"`.

For the test `pruebas ./spec/requests/graphql/queries/connections/appointments_query_spec.rb:528`, the errors I'm getting I suspect are related to this association not being properly created. Let's see.

When inspecting `#tags` at `app/models/appointment.rb`
```ruby
def tags
  # (...)
  ap number
  ap episode.latest_reauthorization_visit_number

  # (...)
end
```

it outputs `nil` for `latest_reauthorization_visit_number`:
```
1
nil

2
nil

3
nil
```

There's no latest reauthorization even though in tests it is setup:
```ruby
# spec/requests/graphql/queries/connections/appointments_query_spec.rb:457

create(:payer_authorization, episode: patient_episode,
		 request_status: :granted,
		 required_gap_in_days: 5,
		 reauthorization_visit_number: 1,
		 visit_amount: 2,
		 submitted_at: Date.new(2018, 10, 14))
```

However, if I inspect how many authorizations the episode has:
```ruby
"este eppisode tiene 1 auths"
```

But, when trying to inspect authorizations at:
```ruby
# app/models/episode.rb

def latest_reauthorization_visit_number
	Rails.cache.fetch("#{cache_key_with_version}/latest_reauthorization_visit_number", expires_in: 5.minutes) do
	  authorizations.to_a.select(&:granted?).max_by(&:reauthorization_visit_number)&.reauthorization_visit_number
	end
end
```

the output is wrapped in a BatchLoader instance:
```
(byebug) authorizations
#<ActiveRecord::Associations::CollectionProxy [#<BatchLoader:0x727800>]>
(byebug) authorizations.to_a
[#<BatchLoader:0x727800>]
authorizations.to_a.select(&:granted?)
[]
```

In this way I get zero authorizations and it changes the output of the tests.

**Something is going on in the batch-loader gem**. I did some changes regarding this gem at [[Exploring ActiveRecord Associations for AppBlend]] but looks like it's not enough.

Why I say this? Because when I access the authorization via direct methods, I can see it exists and is created as described in the tests:
```ruby
(byebug) self.id # Episode
"1879956f-a8c1-49e2-bebc-d0f436a861e9"
(byebug) PayerAuthorization.where(episode_id: "1879956f-a8c1-49e2-bebc-d0f436a861e9")
#<ActiveRecord::Relation [#<PayerAuthorization id: "b771918c-e16d-4f9e-ab6d-a91d2c38280b", visit_amount: 2, authorization_number: "FAHXSJNMCE", effective_until: "2019-04-14", reauthorization_visit_number: 1, required_gap_in_days: 5, request_status: "granted", episode_id: "1879956f-a8c1-49e2-bebc-d0f436a861e9", created_at: "2018-10-14 11:00:00.000000000 +0000", updated_at: "2018-10-14 11:00:00.000000000 +0000", deleted_at: nil, submitted_at: "2018-10-14">]>
```

### Batch Loader in Rails 6.1

This is the output of exploring the instances when they reach batch-loader method_missing method:
```
Veamos que es method_name: authorizations
"este eppisode tiene 1 auths"
Veamos que es sync: Episode
Veamos que es method_name: latest_reauthorization_visit_number
"Veamos que es authorizations: ActiveRecord::Associations::CollectionProxy"
Veamos que es sync: Array
Veamos que es method_name: length
Veamos que es sync: Array
Veamos que es method_name: each
Veamos que es sync: Array
Veamos que es method_name: size
[
    [0] #<PayerAuthorization:0x000000013b816618> {
                                  :id => "4b84a751-89b6-40cb-b852-a425c1e231c2",
                        :visit_amount => 2,
                :authorization_number => "BDQYEKHLFZ",
                     :effective_until => Sun, 14 Apr 2019,
        :reauthorization_visit_number => 1,
                :required_gap_in_days => 5,
                      :request_status => "granted",
                          :episode_id => "e83354a6-3500-4c47-a9f4-fde381e7b3fb",
                          :created_at => Sun, 14 Oct 2018 11:00:00.000000000 UTC +00:00,
                          :updated_at => Sun, 14 Oct 2018 11:00:00.000000000 UTC +00:00,
                          :deleted_at => nil,
                        :submitted_at => Sun, 14 Oct 2018
    }
]
Veamos que es sync: Array
Veamos que es method_name: dup
"Tiene Latest reauth: 1"
Veamos que es sync: Episode
Veamos que es method_name: latest_reauthorization_visit_number
Veamos que es sync: Episode
Veamos que es method_name: should_trigger_reauthorization?
Veamos que es sync: Array
Veamos que es method_name: each
Veamos que es sync: Episode
Veamos que es method_name: total_expected_visits
Veamos que es sync: Episode
Veamos que es method_name: authorizations
"este eppisode tiene 1 auths"
Veamos que es sync: Episode
Veamos que es method_name: latest_reauthorization_visit_number
"Tiene Latest reauth: 1"
Veamos que es sync: Episode
Veamos que es method_name: latest_reauthorization_visit_number
Veamos que es sync: Episode
Veamos que es method_name: total_expected_visits
Veamos que es sync: Episode
Veamos que es method_name: authorizations
"este eppisode tiene 1 auths"
Veamos que es sync: Episode
Veamos que es method_name: latest_reauthorization_visit_number
"Tiene Latest reauth: 1"
Veamos que es sync: Episode
Veamos que es method_name: latest_reauthorization_visit_number
Veamos que es sync: Episode
Veamos que es method_name: load_many
Veamos que es sync: Array
Veamos que es method_name: each
Veamos que es sync: Episode
Veamos que es method_name: present?
Veamos que es sync: Episode
Veamos que es method_name: load_many
Veamos que es sync: Array
Veamos que es method_name: each
Veamos que es sync: Episode
Veamos que es method_name: present?
Veamos que es sync: Episode
Veamos que es method_name: load_many
Veamos que es sync: Array
Veamos que es method_name: each
Veamos que es sync: Episode
Veamos que es method_name: present?
Veamos que es sync: Episode
Veamos que es method_name: authorizations
"este eppisode tiene 1 auths"
Veamos que es sync: Episode
Veamos que es method_name: latest_reauthorization_visit_number
"Tiene Latest reauth: 1"
Veamos que es sync: Episode
Veamos que es method_name: latest_reauthorization_visit_number
Veamos que es sync: Episode
Veamos que es method_name: should_trigger_reauthorization?
Veamos que es sync: Array
Veamos que es method_name: each
Veamos que es sync: Episode
Veamos que es method_name: total_expected_visits
Veamos que es sync: Episode
Veamos que es method_name: authorizations
"este eppisode tiene 1 auths"
Veamos que es sync: Episode
Veamos que es method_name: latest_reauthorization_visit_number
"Tiene Latest reauth: 1"
Veamos que es sync: Episode
Veamos que es method_name: latest_reauthorization_visit_number
Veamos que es sync: Episode
Veamos que es method_name: total_expected_visits
Veamos que es sync: Episode
Veamos que es method_name: authorizations
"este eppisode tiene 1 auths"
Veamos que es sync: Episode
Veamos que es method_name: latest_reauthorization_visit_number
"Tiene Latest reauth: 1"
Veamos que es sync: Episode
Veamos que es method_name: latest_reauthorization_visit_number
```

Notice how at any point it is sending a `method_name` being `new_record?` Which is the reason it fails in Rails 7.0.4:
```
NoMethodError:
       undefined method `new_record?' for [#<PayerAuthorization>]:Array
```

### Update

Reason, apparently, found at [[Browsing Batch Loader in Rails 7.0.4#A workaround // fix]]