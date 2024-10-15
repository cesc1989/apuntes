# Upgrade Edge to 7.0.8.4

# ActionView::MissingTemplate in Admin::PatientDuplicates#compare ✅

En Patient Duplicates al clicar "Compare & Resolve".

```bash
Showing /Users/francisco/projects/luna-project/backend/app/views/admin/patient_duplicates_report/compare.html.erb where line #26 raised:

Missing partial admin/patient_duplicates/_patient_information, active_admin/resource/_patient_information, active_admin/base/_patient_information, inherited_resources/base/_patient_information, application/_patient_information with {:locale=>[:en], :formats=>[:html], :variants=>[], :handlers=>[:raw, :erb, :html, :builder, :ruby, :coffee, :arb, :haml]}.

Searched in:
  * "/Users/francisco/projects/luna-project/backend/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/rails_email_preview-2.2.3/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/graphiql-rails-1.9.0/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/devise_token_auth-1.2.1/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/devise-security-0.17.0/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/devise-4.8.1/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/activeadmin-2.14.0/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/kaminari-core-1.2.2/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/actiontext-7.0.8.4/app/views"
  * "/Users/francisco/.gem/ruby/3.1.0/gems/actionmailbox-7.0.8.4/app/views"

Did you mean?  admin/patient_duplicates_report/patient_information
               active_admin/base/bonus_form
               admin/patients/patient_outstanding_items
               admin/patients/waive_form
               admin/patients/info
               admin/patients/edit_appointment_button
```

En
```ruby
<div class="patient-dup-container">
    <div class="p1 patient-container <%= "selected" if suggested_patient(@patient1, @patient2) == @patient1 %>">
    
      <%= render "patient_information", patient: @patient1 %>

      <% unless antimatch_exists? || merge_exists? || !can_merge?(@patient1, @patient2) %>
        <div class="patient-info-section">
          <label class="patient-info-header">Actions</label>
```

~~Por alguna razón, en Rails 7.0.8.4~~ Este es un error viejo pasando desde Rails 6.1. Parece que nadie usa esta característica. El error es porque no se puede encontrar ese partial que está en la misma carpeta de la vista que lo llama.

Me tocó poner la ruta completa:
```ruby
<%= render partial: "admin/patient_duplicates_report/patient_information", locals: { patient: @patient1 } %>
```

Puede ser porque hay muchos partials en el sistema con el mismo nombre:
```
Missing partial admin/patient_duplicates/_patient_information, active_admin/resource/_patient_information, active_admin/base/_patient_information, inherited_resources/base/_patient_information, application/_patient_information
```

# Cambios de precision: 6/nil en schema para campos datetime

## Contexto

Desde Rails 6.0, a los campos datetime de los timestamps (created_at, updated_at) se les imprime el valor para precision en el schema:
```ruby
t.datetime "created_at", precision: nil, null: false
t.datetime "updated_at", precision: nil, null: false
```

> [!INFO]
> Este es [el commit](https://github.com/rails/rails/commit/57015cdfa2083351f64a82f7566965172a41efcb) donde se configura el precision para los timestamps.
> 
> Y este es el [changelog](https://github.com/rails/rails/blob/6-0-stable/activerecord/CHANGELOG.md) de la versión 6.x.x

Cuando hice el upgrade a Rails 7.0.7, muchos de estos campos quedaron mostrando el precision en el schema:
```diff
- t.datetime "created_at", null: false
+ t.datetime "created_at", precision: nil, null: false
```

Sin embargo, ahora que estoy haciendo el upgrade a Rails 7.0.8.4, en el schema a varios timestamps se le está quitando el precision:
```diff
-    t.datetime "created_at", precision: 6, null: false
-    t.datetime "updated_at", precision: 6, null: false
+    t.datetime "created_at", null: false
+    t.datetime "updated_at", null: false
```

¿Por qué pasa esto?

## Conclusiones

Esta [respuesta en Stack Overflow](https://stackoverflow.com/a/71482301/1407371) explica toda la historia de este cambio.

La conclusión es que desde Rails 7.0.2 ya no debe reflejarse el valor de precisión para los timestamps en el schema porque se maneja de manera interna. Así [lo comentan](https://github.com/rails/rails/issues/44571#issuecomment-1059295012) en issue donde se ve esta situación:

![[schema.no.precision.in.timestamps.png]]