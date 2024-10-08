# Upgrade to Rails 7.0.x Notes 2 ✅

This is a continuation of [[Upgrade to Rails 7.0.x Notes ✅]]

# Unsafe redirect to "https://www.getluna.com", pass allow_other_host: true to redirect anyway ✅

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

# Value not defined un Enum raises ActiveRecord::NotNullViolation with null constraint ✅

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

Fixed by setting the default enum value in the factory of patient serviceability check.

# Error with PayerAuthorization and BatchLoader ✅

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

**Something is going on in the batch-loader gem**. I did some changes regarding this gem at [[007 - ✅ Exploring ActiveRecord Associations for AppBlend]] but looks like it's not enough.

Why I say this? Because when I access the authorization via direct methods, I can see it exists and is created as described in the tests:
```ruby
(byebug) self.id # Episode
"1879956f-a8c1-49e2-bebc-d0f436a861e9"
(byebug) PayerAuthorization.where(episode_id: "1879956f-a8c1-49e2-bebc-d0f436a861e9")
#<ActiveRecord::Relation [#<PayerAuthorization id: "b771918c-e16d-4f9e-ab6d-a91d2c38280b", visit_amount: 2, authorization_number: "FAHXSJNMCE", effective_until: "2019-04-14", reauthorization_visit_number: 1, required_gap_in_days: 5, request_status: "granted", episode_id: "1879956f-a8c1-49e2-bebc-d0f436a861e9", created_at: "2018-10-14 11:00:00.000000000 +0000", updated_at: "2018-10-14 11:00:00.000000000 +0000", deleted_at: nil, submitted_at: "2018-10-14">]>
```

### Batch Loader in Rails 6.1 👍🏽

**Update**: Reason, found at [[Browsing Batch Loader in Rails 7.0.4 ✅#A workaround // fix]]

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
```

Notice how at any point it is sending a `method_name` being `new_record?` Which is the reason it fails in Rails 7.0.4:
```
NoMethodError:
       undefined method `new_record?' for [#<PayerAuthorization>]:Array
```

# undefined method has_key? for nil:NilClass encrypted_attributes ✅

Got this one after running `bundle exec rails app:update` and when checking Zeitwerk was ok:
```bash
$ bundle exec rails zeitwerk:check --trace
** Invoke zeitwerk:check (first_time)
** Invoke environment (first_time)
** Execute environment
rails aborted!
NoMethodError: undefined method `has_key?' for nil:NilClass

    encrypted_attributes.has_key?(attribute.to_sym)
                        ^^^^^^^^^
/Users/francisco/.gem/ruby/3.1.0/gems/attr_encrypted-3.1.0/lib/attr_encrypted.rb:224:in `attr_encrypted?'
/Users/francisco/.gem/ruby/3.1.0/gems/devise-two-factor-4.0.2/lib/devise_two_factor/models/two_factor_authenticatable.rb:17:in `block in <module:TwoFactorAuthenticatable>'
/Users/francisco/.gem/ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/concern.rb:136:in `class_eval'
/Users/francisco/.gem/ruby/3.1.0/gems/activesupport-7.0.4/lib/active_support/concern.rb:136:in `append_features'
/Users/francisco/.gem/ruby/3.1.0/gems/devise-4.8.1/lib/devise/models.rb:105:in `include'
/Users/francisco/.gem/ruby/3.1.0/gems/devise-4.8.1/lib/devise/models.rb:105:in `block (2 levels) in devise'
```

> Why didn't this show in my previous work branch?

The fix is to update devise-two-factor gem to 4.1.1.

Found mention of this error in these links:

- https://github.com/attr-encrypted/attr_encrypted/issues/423
- https://github.com/attr-encrypted/attr_encrypted/pull/434

# Incompatible marshal file format

Error:
```ruby
TypeError

incompatible marshal file format (can't be read)
	format version 4.8 required; 0.4 given
```

Caught in this line:
```ruby
Rails.cache.fetch("#{cache_key_with_version}/total_expected_visits", expires_in: 24.hours, skip_nil: true) do
```

In [this Stack Overflow](https://stackoverflow.com/questions/23629879/incompatible-marshal-file-format-cant-be-read-format-version-4-8-required-0) answers suggest it might be a cache problem when changing Rails versions.

Possible fixes:

- bundle exec rake assets:clean then bundle exec rake assets:precompile
- change the secret key base to invalidate all sessions

In the Sentry report the Rails gems version were on the current version 6.1.7.8.

# ActionView::Template::Error ✅

Este error:
```ruby
undefined method episode_path for #<ActionView::Base:0x00000000254ba8>

              target.public_send(method, *args)
                    ^^^^^^^^^^^^
Did you mean?  paid_path
```

Pasa en esta parte:
```ruby
    <%= f.inputs "Insurance Details" do %>
```

Lo cual no tiene mucho sentido así que debe ser algo de Rails 7. Así se ve el stack trace en sentry:
```
Crashed in non-app: actionpack (7.0.7) lib/action_dispatch/routing/polymorphic_routes.rb in public_send

app/views/admin/care_plans/_form.html.erb at line 36

<%= f.inputs "Insurance Details" do %>

Called from: actionview (7.0.7) lib/action_view/helpers/capture_helper.rb in block in capture

app/views/admin/care_plans/_form.html.erb at line 15

<%= semantic_form_for [:admin, @care_plan], url: @care_plan.new_record? ? admin_care_plans_path : admin_care_plan_path(@care_plan) do |f| %>
```

**Enlaces**
- Relacionado: [Problems with URL generation](https://github.com/rails/rails/issues/45331)

## Contexto en las vistas

Los reportes de Sentry indican estas rutas.

En local:
```
http://localhost:3000/admin/care_plans/becc35e0-bd3b-4f35-858c-f0acd27f2415/edit
```

En Alpha:
```
https://luxe.alpha.getluna.com/admin/care_plans/700d95a1-8163-4c39-92e3-35ead33b8e82/edit
```

En Omega:
```
https://luxe.getluna.com/admin/care_plans/1532a394-0eae-459f-989d-a41cd17494b0/edit
```

Se llega desde el perfil del paciente. Cuando paciente tiene Care Plan, aparecen los botones:
- Create New Care Plan
- Edit Care Plan
- Edit Authorizations

Al clicar el segundo enlace llegamos a la pantalla donde se genera el error.

### ¿Dónde se definen la ruta?

Ruta: `edit_admin_care_plan_path`.

En el admin Patient, se renderiza el partial recent_care_plan:
```ruby
if patient.episodes.any?
          render partial: "recent_care_plan", locals: {
            patient: patient,
            recent_care_plan: patient.recent_care_plan,
            recent_draft_care_plan: patient.recent_draft_care_plan
          }
```

Dicho partial tiene los enlaces a los botones:
```ruby
<%= link_to "Edit Care Plan", edit_admin_care_plan_path(recent_care_plan), class: "admin_btn" %>
```

## Error con best_in_place?

Esta gema https://github.com/bernat/best_in_place

En local, el stack trace me lleva hasta esta línea:
```ruby
<%= best_in_place Episode.maybe_draft.find(f.object.id), :billing_notes, as: :textarea, url: f.options[:url] %>
```

Así se ve el trace:
```
actionpack (7.0.7) lib/action_dispatch/routing/polymorphic_routes.rb:280:in public_send

actionpack (7.0.7) lib/action_dispatch/routing/polymorphic_routes.rb:280:in handle_model_call

actionview (7.0.7) lib/action_view/routing_url_for.rb:119:in url_for

best_in_place (88eb3052623a) lib/best_in_place/helper.rb:51:in best_in_place
app/views/admin/care_plans/_form.html.erb:134 <== AQUI
```

Eso tendría más sentido.

Comenté esa línea y subí a alfa. No explota la página de edición de Care Plan. Es en esta parte donde se usa esa gema pero puede ser la construcción de la URL.

`best_in_place` se carga en estas páginas:

En `app/views/admin/care_plans/_form.html.erb`. Línea 134.

Definición:
```ruby
<%= best_in_place Episode.maybe_draft.find(f.object.id), :billing_notes, as: :textarea, url: f.options[:url] %>
```

En `app/admin/customers/patients.rb`. Línea 1193.

Definición:
```ruby
row :billing_notes do |care_plan|
	best_in_place care_plan, :billing_notes, as: :textarea, url: admin_care_plan_path(care_plan)
end
```

En `app/admin/episodes.rb`. Línea 236.

En esta no da error. Care Plan [show](http://localhost:3000/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9) local.

Definición:
```ruby
row :billing_notes do |care_plan|
	best_in_place care_plan, :billing_notes, as: :textarea, url: admin_care_plan_path(care_plan)
end
```

En `app/admin/payer_management/care_plans.rb`. Línea 358.

En esta no da error. [Listado](http://localhost:3000/admin/care_plan_route_reviews).

Definición:
```ruby
column :billing_notes do |care_plan|
	best_in_place care_plan, :billing_notes, as: :textarea, url: admin_care_plan_path(care_plan)
end
```

## El Problema es f.options[:url] de Formtastic/FormBuilder

Cuando estamos en Rails 6.1.7.8:
```ruby
<%= f.options.inspect %>

{
  :url=>"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9",
  :builder=>Formtastic::FormBuilder,
  :html=> {
    :class=>"formtastic care_plan_form",
    :id=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9",
    :method=>:patch,
    :novalidate=>false,
    :authenticity_token=>nil
  },
  :custom_namespace=>nil
}
```

Si buscamos la URL:
```ruby
<%= f.options[:url].inspect %>

"/admin/care_plans/da175c34-9c95-4e13-9860-f77feb22aad9"
```

Cuando se está en Rails 7.0.7:

```ruby
<%= f.options.inspect %>

{
  :allow_method_names_outside_object=>false,
  :skip_default_ids=>false,
  :builder=>Formtastic::FormBuilder,
  :html=> {
    :class=>"formtastic care_plan_form",
    :id=>"edit_care_plan_form_da175c34-9c95-4e13-9860-f77feb22aad9",
    :novalidate=>false
  },
  :custom_namespace=>nil,
  :local=>true
}
```


```ruby
<%= f.options[:url].inspect %>

nil
```
