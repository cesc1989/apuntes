# Pruebas de Webhooks y otros

# Prueba de CompletedForm falla por el orden de los atributos

La prueba en `spec/lib/webhooks/completed_form.rb` falla porque la clase devuelve los atributos en un orden distinto a como se generan en la prueba:
```ruby
# En la clase
'{"form":{"progress_type":"onboarding","injury_name":"InjuryName1","pain_scale":5,"uuid":"b254261a-1769-42ce-84f0-8a14a54eb3fb","completed_at":"2018-10-14T11:00:00.000Z","id":1},"patient":{"id":1,"full_name":"FirstName1 LastName1","internal_id":"58e194a8-9f6c-4de5-a0d4-266deec83f30"}}'

# En la prueba
"{\"form\":{\"id\":1,\"progress_type\":\"onboarding\",\"injury_name\":\"InjuryName1\",\"completed_at\":\"2018-10-14T11:00:00.000Z\",\"uuid\":\"b254261a-1769-42ce-84f0-8a14a54eb3fb\",\"pain_scale\":5},\"patient\":{\"id\":1,\"full_name\":\"FirstName1 LastName1\",\"internal_id\":\"58e194a8-9f6c-4de5-a0d4-266deec83f30\"}}",
```

Para arreglarlo tuve que cambiar el orden del payload en la prueba.

De esto originalmente:
```ruby
let(:payload) do
	{
		"form": {
			"id": form.id,
			"progress_type": form.progress_type,
			"injury_name": form.injury_name,
			"completed_at": form.completed_at,
			"uuid": form.uuid,
			"pain_scale": form.pain_scale
		},
		"patient": {
			"id": patient.id,
			"full_name": patient.full_name,
			"internal_id": patient.internal_id
		}
	}
end
```

A esto modificado:
```ruby
let(:payload) do
	{
		"form": {
			"progress_type": form.progress_type,
			"injury_name": form.injury_name,
			"pain_scale": form.pain_scale,
			"uuid": form.uuid,
			"completed_at": form.completed_at,
			"id": form.id
		},
		"patient": {
			"id": patient.id,
			"full_name": patient.full_name,
			"internal_id": patient.internal_id
		}
	}
end
```

¿Por qué pasa?

# El mensaje de deprecation de `Date#to_s` el formato por defecto (`:default`) 🗓️

> [!Note]
> Esto ya lo vi cuando estaba actualizando Edge a Rails 7 en [[Upgrade to Rails 7.0.x Notes ✅#Rails 7.0 ignores default format for Date and Time ✅]] y también en [[004 - 👌🏽 AppBlend What Changes in Config Defaults#What is config.active_support.disable_to_s_conversion?]]

En los initializers está el archivo `date_time.rb` con esto:
```ruby
# frozen_string_literal: true

# In Rails < 7.0.7 this will not be interpolated by default. One need to explicitly tell
# the date to be formated when interpolating, i.e:
#
#    Date.current.to_fs(:default)
#
# In version 7.0.7 there's a fix to bring back this behavior.
#
# In Rails 7.1 this should be done using i18n instead as recomended by Rafael França -> https://github.com/rails/rails/pull/48555#issuecomment-1680844190
Date::DATE_FORMATS[:default] = "%m/%d/%Y"
Time::DATE_FORMATS[:default] = "%m/%d/%Y %H:%M"

Time::DATE_FORMATS[:patient_self_report] = "%a, %e %b %Y"
Date::DATE_FORMATS[:patient_self_report] = "%Y-%m-%d"
```

Pero esto está causando mucho ruido de depreciaciones al correr las pruebas:
```bash
DEPRECATION WARNING: Using a :default format for Date#to_s is deprecated. Please use Date#to_fs instead. If you fixed all places inside your application that you see this deprecation, you can set `ENV['RAILS_DISABLE_DEPRECATED_TO_S_CONVERSION']` to `"true"` in the `config/application.rb` file before the `Bundler.require` call to fix all the callers outside of your application. (called from block (4 levels) in <top (required)> at /Users/francisco/projects/luna-project/backend/spec/requests/patient_self_report/api/v2/form_status/form_status_is_urgent_spec.rb:40)
```

El mensaje de deprecation se da porque, [en Rails 7.0.8.4](https://github.com/rails/rails/pull/48555#issuecomment-1713935272), se respeta que se use el formato por defecto de Date o Time pero en todo caso se muestra el mensaje para alertar a los devs:

> My understanding is that this PR fixes [#48545](https://github.com/rails/rails/issues/48545) by making `to_s` respect a defined `:default` format _while producing a deprecation warning_.
>
> (Meaning that every call to `to_s`, like those from Active Job and exception_notification, ==will see that warning as long as the application has a `:default` format defined==. And that [at least IMO] the implied resolution is to not set a value for `:default`.)

En el resalto pongo lo que creo es la causa principal. Tener un formato `default` definido.

## Soluciones

### Usar i18n

En [este comentario](https://github.com/rails/rails/pull/48555#issuecomment-1680844190) se dice que se debe usar i18n:

> Yes this is intended. *Without this deprecation, you would get a very different format when upgrading to 7.1.* I usually **recommend people to use i18n to change the format of Date and Time objects** instead of changing the `:default` value (...).

### Quitar el formato por defecto 🎉

Así probé:
```diff
- Date::DATE_FORMATS[:default] = "%m/%d/%Y"
- Time::DATE_FORMATS[:default] = "%m/%d/%Y %H:%M"
+ Date::DATE_FORMATS[:luxe] = "%m/%d/%Y"
+ Time::DATE_FORMATS[:luxe] = "%m/%d/%Y %H:%M"
```

Y se fueron los mensajes de depreciación.

> [!important]
> Esto funciona pero tengo que probarlo con todas las pruebas en general y el sistema en general.


# ⚠️ Fecha fijada en Octubre 2018 causa problemas con fecha alterando el orden de los registros ⚠️

Esta configuración en `rails_helper`:
```ruby
Rails.application.config.time_zone = "Pacific Time (US & Canada)"
Timecop.freeze(DateTime.parse("2018-10-14T11:00:00 +0000"))
```

Hace que todas las fechas queden en ese día con la misma hora, minutos y segundos.

Esto hace que el orden por `created_at` quede anulado.

## Ejemplos

En la prueba para `api/v2/form_status/form_status_is_urgent_spec.rb` así se veían las fechas a comparar:
```ruby
(byebug) DateTime.parse(next_appt)
Mon, 15 Oct 2018 00:00:00 +0000

(byebug) acceptable_remaining_time_minutes.from_now
Mon, 15 Oct 2018 11:00:00.000000000 UTC +00:00
```

Entonces la prueba fallaba. Para poder arreglarlo tocó agregarle horas a una de las fechas:
```ruby
let(:next_appointment) { Time.zone.today + 72.hours }
```

También pasaba en `api/v3/forms/aggravating_activities_spec.rb`. Aquí, en la prueba `it "lists all three aggravating_activities"` contaba con traer las aggravating activities en orden de creación pero no pasa porque todas tienen la misma fecha y hora:
```ruby
[
    [0] #<PatientSelfReport::AggravatingActivity:0x0000000114858990> {
                :id => 1,
           :form_id => 1,
              :name => "Activity1",
        :created_at => Sun, 14 Oct 2018 11:00:00.000000000 UTC +00:00,
        :updated_at => Sun, 14 Oct 2018 11:00:00.000000000 UTC +00:00,
    },
    [1] #<PatientSelfReport::AggravatingActivity:0x0000000114853af8> {
                :id => 2,
           :form_id => 1,
              :name => "Activity2",
        :created_at => Sun, 14 Oct 2018 11:00:00.000000000 UTC +00:00,
        :updated_at => Sun, 14 Oct 2018 11:00:00.000000000 UTC +00:00,
    },
    [2] #<PatientSelfReport::AggravatingActivity:0x00000001148531c0> {
                :id => 3,
           :form_id => 1,
              :name => "Activity3",
        :created_at => Sun, 14 Oct 2018 11:00:00.000000000 UTC +00:00,
        :updated_at => Sun, 14 Oct 2018 11:00:00.000000000 UTC +00:00,
    }
]
F
```

Me tocó hacer esto:
```diff
- form = FactoryBot.create(:form, :with_aggravating_activities)
+ form = FactoryBot.create(:form)
+ activity_1 = FactoryBot.create(:aggravating_activity, form: form, created_at: Time.zone.now)
+ activity_2 = FactoryBot.create(:aggravating_activity, form: form, created_at: Time.zone.now + 2.hours)
+ activity_3 = FactoryBot.create(:aggravating_activity, form: form, created_at: Time.zone.now + 3.hours)
```

para poder controlar la hora de cada una.

# Caso JavaScript BigInt: agrega digitos adicionales

Etiquetas: #javascript #bigint

Al probar guardar un form desde la UI ocurría que en el payload de la petición se agregaban dígitos adicionales al ID de medication o surgery:
```ruby
# Original
[1120861213391190272, 9021913502528152935, 3791503591677988816]

# Modified by JS
1120861213391190300
9021913502528153000
3791503591677989000
```

Nota el 300, 3000 y 9000 al final. Al parecer esto es un caso de limitaciones de JavaScript, según comentan en [este issue](https://github.com/josdejong/jsoneditor/issues/231#issuecomment-148649072):
> this is a limitation in the Number format of JavaScript, which has a precision of about 16 digits.