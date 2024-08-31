# Upgrade to Rails 7.0.4 - Notes Part 2

This is a continuation of [[Upgrade to Rails 7.0.4 - Notes]]

# Errors in CI

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
