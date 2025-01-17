# Fix default format for DATETIME - Notes

This needs to be fixed so that the deprecation message goes away:

```
DEPRECATION WARNING: Using a :default format for TimeWithZone#to_s is deprecated. Please use TimeWithZone#to_fs instead. If you fixed all places inside your application that you see this deprecation, you can set `ENV['RAILS_DISABLE_DEPRECATED_TO_S_CONVERSION']` to `"true"` in the `config/application.rb` file before the `Bundler.require` call to fix all the callers outside of your application.
```

# Context

One way to go about this is by removing the `:default` format altogether from the initializer. However, in Edge, by doing this form inputs are also affected and it's more difficult to handle.

We got this default in `config/initializers/date_time.rb`
```ruby
Date::DATE_FORMATS[:default] = "%m/%d/%Y"
Time::DATE_FORMATS[:default] = "%m/%d/%Y %H:%M"
```

Changing it to other than `default` will break some forms and will make dates to be printed with a different format.
```ruby
Date::DATE_FORMATS[:luxe] = "%m/%d/%Y"
Time::DATE_FORMATS[:luxe] = "%m/%d/%Y %H:%M"
```

So, what I'm doing for the AppBlend efforts is to go to every place where a date interpolation is happening and change it there. The change is to replace `to_s` with `to_fs` or add `to_fs` in string interpolations. 

There are some ways to find those places:

- Look for `to_s` calls
- Use this regex in the code editor finder: `\w+_(at|from|until)(&?\.to_date)?(\}|&?\.to_s)`

# In String Interpolations append `to_fs`

In `#{}` string interpolation, add `to_fs` to the date instance:
```ruby
[3] pry(main)> ap "hola: #{Availability.first.created_at}"
  Availability Load (0.6ms)  SELECT "availabilities".* FROM "availabilities" ORDER BY "availabilities"."id" ASC LIMIT $1  [["LIMIT", 1]]
DEPRECATION WARNING: Using a :default format for TimeWithZone#to_s is deprecated. Please use TimeWithZone#to_fs instead. If you fixed all places inside your application that you see this deprecation, you can set `ENV['RAILS_DISABLE_DEPRECATED_TO_S_CONVERSION']` to `"true"` in the `config/application.rb` file before the `Bundler.require` call to fix all the callers outside of your application. (called from <main> at bin/rails:4)
"hola: 09/06/2018 16:04"
=> nil
[4] pry(main)> ap "hola: #{Availability.first.created_at.to_fs}"
  Availability Load (0.8ms)  SELECT "availabilities".* FROM "availabilities" ORDER BY "availabilities"."id" ASC LIMIT $1  [["LIMIT", 1]]
"hola: 09/06/2018 16:04"
```

# Replace `to_date` with DateTimeLocalizer

Replacing the `:default` to `:luxe` makes some test break:
```bash
Failure/Error:
       expect(data[:node][:carePlans]).to eq(
         [
           name: "#{episode.injury.name} (#{appt1.scheduled_date.to_date} - #{appt2.scheduled_date.to_date})"
         ]
       )

       expected: [{:name=>"injury159664 (2018-10-07 - 2018-10-21)"}]
            got: [{:name=>"injury159664 (10/07/2018 - 10/21/2018)"}]
```

When using `to_fs(:luxe)` instead of `to_date` still does not work:
```bash
Failure/Error:
       expect(data[:node][:carePlans]).to eq(
         [
           name: "#{episode.injury.name} (#{appt1.scheduled_date.to_fs(:luxe)} - #{appt2.scheduled_date.to_fs(:luxe)})"
         ]
       )

       expected: [{:name=>"injury161094 (10/07/2018 11:00 - 10/21/2018 11:00)"}]
            got: [{:name=>"injury161094 (10/07/2018 - 10/21/2018)"}]
```

It's using the format in Time constant.

## Is this code using the default format?

Without the `to_date` the printed date is:
```ruby
Sun, 07 Oct 2018 11:00:00.000000000 UTC +00:00
```

With the `to_date`:
```ruby
Sun, 07 Oct 2018
```

## What is `to_date`?

[Docs](https://apidock.com/rails/String/to_date).

> Converts a string to a Date value.

```ruby
"1-1-2012".to_date   # => Sun, 01 Jan 2012
"01/01/2012".to_date # => Sun, 01 Jan 2012
"2012-12-13".to_date # => Thu, 13 Dec 2012

"12/13/2012".to_date # => ArgumentError: invalid date
```

## How to fix?

Use the `DateTimeLocalizer` class with `american` method. For example:
```ruby
DateTimeLocalizer.new(appt1.scheduled_date, appt1.region).american
#=> 10/07/2018
```

# When using `to_date` append `to_fs`

Another way to go around places using `to_date` is to append `to_fs`:
```ruby
[2] pry(main)> "#{Workout.first.created_at.to_date}"
  Workout Load (0.9ms)  SELECT "workouts".* FROM "workouts" ORDER BY "workouts"."id" ASC LIMIT $1  [["LIMIT", 1]]
DEPRECATION WARNING: Using a :default format for Date#to_s is deprecated. Please use Date#to_fs instead. If you fixed all places inside your application that you see this deprecation, you can set `ENV['RAILS_DISABLE_DEPRECATED_TO_S_CONVERSION']` to `"true"` in the `config/application.rb` file before the `Bundler.require` call to fix all the callers outside of your application. (called from <main> at bin/rails:4)
=> "12/15/2023"
[3] pry(main)> "#{Workout.first.created_at.to_date.to_fs}"
  Workout Load (0.9ms)  SELECT "workouts".* FROM "workouts" ORDER BY "workouts"."id" ASC LIMIT $1  [["LIMIT", 1]]
=> "12/15/2023"
```

# Fixing Edit forms date inputs

In Forms, or at least in Active Admin (using Formtastic) forms, the date input fields seem to be using the default date format by using the `to_s` method. To fix this, I'm going through all forms and checking whether they're being affected.

In affected forms with date fields, we can see the value being scrambled:
```ruby
# good date
01/10/2025

# bad, affected date
20/25/0110
```

## Form Inputs

In form inputs, add the `value` property to call the `to_fs` method to the date attribute.

```diff
- f.input :submitted_at, as: :string, input_html: { class: "date" }
+ f.input :submitted_at, as: :string, input_html: { class: "date", value: f.object.submitted_at&.to_fs }
```

## best_in_place gem/library

For places using `best_in_place` [library](https://github.com/mmotherwell/best_in_place), pass the `value` attribute to the method call.

```diff
best_in_place(
  care_plan,
  :surgery_date,
  as: :date,
  url: "/admin/pathway_assignments/#{care_plan.id}",
  place_holder: "Unset",
+ value: care_plan.surgery_date&.to_fs
)
```

# Errors originating from gems or libraries

Even after fixing problems in places interpolating string with `to_fs` calls, some libraries still use that method and it creeps out to the application log.

For example, the Pathway Assignments index view uses `best_in_place` to edit two date fields. This is the log after loading that view:
```
DEPRECATION WARNING: Using a :default format for Date#to_s is deprecated. Please use Date#to_fs instead. If you fixed all places inside your application that you see this deprecation, you can set `ENV['RAILS_DISABLE_DEPRECATED_TO_S_CONVERSION']` to `"true"` in the `config/application.rb` file before the `Bundler.require` call to fix all the callers outside of your application. (called from best_in_place_build_value_for at /Users/francisco/.gem/ruby/3.1.0/bundler/gems/best_in_place-88eb3052623a/lib/best_in_place/helper.rb:121)
```

Line 121 in [best_in_place code has this](https://github.com/bernat/best_in_place/blob/master/lib/best_in_place/helper.rb#L121):
```ruby
object.send(field).to_s
```

In this test run, it originates from a factory:
```
DEPRECATION WARNING: Using a :default format for Date#to_s is deprecated. Please use Date#to_fs instead. If you fixed all places inside your application that you see this deprecation, you can set `ENV['RAILS_DISABLE_DEPRECATED_TO_S_CONVERSION']` to `"true"` in the `config/application.rb` file before the `Bundler.require` call to fix all the callers outside of your application. (called from block (3 levels) in <top (required)> at /usr/src/app/spec/factories/patients.rb:7
```

Do we need to also go and fix those? Maybe no. Read this carefully:

> If you fixed all places inside your application that you see this deprecation, you can set `ENV['RAILS_DISABLE_DEPRECATED_TO_S_CONVERSION']` to `"true"` in the `config/application.rb` file before the `Bundler.require` call to ==fix all the callers outside of your application.==

Emphasis on

> _to fix all the callers outside of your application._

So by setting this ENV we can remove the noise generated by external libraries such as best_in_place or factory_bot.

# Problemas con best_in_place

Para el uso en /admin/credentialing_entries no está funcionando el bip. De hecho, ni siquiera carga. Esto parece ser porque la clase `.date` está aplicando una máscara y está haciendo conflicto.

Así sale el HTML con la clase `.date`:

```html
<td class="col col-effective_from">
  <div class="col-credentialing_entry_effective_from editable date">20/18/0906</div>
</td>
```

Cuando quito esa clase, el campo editable para bip sí carga el html esperado:

```html
<td class="col col-effective_from">
  <div class="col-credentialing_entry_effective_from editable">
    <span
      data-bip-type="date"
      data-bip-attribute="effective_from"
      data-bip-placeholder="Unset"
      data-bip-object="credentialing_entry"
      data-bip-original-content="09/06/2018"
      data-bip-skip-blur="false"
      data-bip-url="/admin/credentialing_entries/974"
      data-bip-value="09/06/2018"
      class="best_in_place"
      id="best_in_place_credentialing_entry_974_effective_from"
    >
      09/06/2018
    </span>
  </div>
</td>
```

Sin embargo, al quitar la clase `.date` el date picker funciona raro.

> [!Note]
> En todo caso con o sin la clase `.date` cuando quito la ENV para que se vaya el mensaje de deprecation, el bip funciona normal.

## ¿Cuál es el fix aquí?

Tengo estas opciones:

- Echar atrás el cambio del ENV que desactiva el deprecation
- Quitar la clase `.date` y luego buscar que funcione bien
- Cambiar la fuente de bip por otro fork

# Problema en fecha exportada desde BD hacía a Athena

El export que ocurre en PatientSummaryWriterWorker para datos que necesita Athena para el Clinical Dashboard cambió el formato de la fecha de Escalation. Al menos, parece que solo esa.

Antes, el export sacaba la fecha `escalated_at` en el formato "MM/DD/YYYY". Hoy, luego de mandar a Omega el cambio de default format lo exportó de esta otra "YYYY-MM-DD". ¿Por qué hace este cambio? ¿No sale directo desde la BD?

Según ChatGPT, en todo caso ActiveRecord está haciendo lo suyo y por eso toma el formato.

Me puse a buscar si había algún export viejo con esa fecha para comprobar que saliera en el formato viejo y así ocurría.

![[01.escalated_american_format.png]]

Luego busqué otro para ver si los otros campos de fecha se verían afectados. Parece que no por un export de otra cosa (sobre visit plans). Así se veía un campo de fecha:

![[02.visit_date_normal_format.png]]

Quedará esperar a que salga algún otro reporte y ver si se dañaron más fechas.

# QA List

Los elementos a revisar cada que haga un cambio con respecto a esta situación.

- edit de benefits_verifications
	- se llama PatientProspect
	- campo: `date_of_birth`
    - Abrir -> `admin/benefit_verifications/b50f8aa2-ba06-4b27-9cc4-34d130830ce2/edit`
- edit de payer authorizations
    - se llega desde "Billing & Finance" -> "Authorizations Report"
    - o acá en `/admin/payer_authorizations`
    - campos: `effective_until` y `submitted_at`
    - Abrir -> `admin/payer_authorizations/47c373cf-8cbd-4ebb-8639-3faa244617d6/edit`
- edit de physicians
	- Al abrir el form edit debe cargar haya o no escalation o portal recipients
	- Abrir -> `admin/physicians/ffd12d73-5af8-438f-9815-797575e7ff21/edit`
- edit care plans
	- Llega desde perfil del paciente -> edit care plan
	- Campo: `effective_until` en Scheduling Limits
	- Campo: `surgery_date`. Hay que seleccionar primero el dropdown que pregunta por un surgery para que se muestre el campo.
    - Abrir -> `admin/care_plans/90d5fb1b-702b-4d36-9292-4e68d61643c3/edit`
- pathway assignments
	- Vista index
	- Campo: `surgery_date`
	- Llega desde Clinical -> Pathway Assignments
	- Abrir -> `admin/pathway_assignments`
- protocol escalations
	- Vista: index
	- Campo: `visit_date`
	- Llega desde Clinical -> Protocol Escalations
	- Abrir -> `admin/protocol_escalations`
- care_plan_route_reviews
	- Llega desde: Payer Management -> Care Plan Route reviews
	- Columna: IV
	- Abrir -> `admin/care_plan_route_reviews`
- ~~Luna Fax Numbers~~
	- ~~Campos: `created_at` y `updated_at`~~
	- ~~Llega desde Clinical -> Luna Fax Numbers~~
	- ~~Abrir -> `admin/luna_fax_numbers`~~
	- Esto fue eliminado de Luxe.
- admin/credentialing_entries
	- Vista index
	- Campo: `effective_from`
	- Abrir -> `admin/credentialing_entries`
- admin/clinic_payers/:id
	- Campos: `effective_from` en la sección Credentialing Entries (si las tiene)
    - Abrir -> `admin/clinic_payers/312836`
