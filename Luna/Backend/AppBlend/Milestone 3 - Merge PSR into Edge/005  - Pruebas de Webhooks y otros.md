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