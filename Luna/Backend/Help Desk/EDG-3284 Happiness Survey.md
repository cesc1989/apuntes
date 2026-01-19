# EDG-3284 Happiness Survey aparece muy seguido

Etiquetas: #luna_help_desk 

Parece que el error se da al:
> Survey completion, log in same day and receive the survey again for certain PTs.

## ¿Qué es esta encuesta?

Parece que es la encuesta de las caras triste, neutral y feliz que sale cuando se inicia sesión en la aplicación como Therapist.

El controlador que la gestiona es `app/controllers/api/v1/therapist/happiness_surveys_controller.rb`.

El modelo es `HappinessSurvey`.

## Therapist de Prueba en Alpha

Therapist con el correo `francisco.quintero+t@ideaware.co`
```ruby
t = Account.find_by(email: "francisco.quintero+t@ideaware.co").therapist
```

## ¿Cuándo se encuesta?

En backend hay dos banderas para controlar cuándo mostrar la survey al therapist.

En el modelo Therapist:
```ruby
def should_prompt_for_happiness_survey?
	return true if last_happiness_survey_prompted_at.nil?
	return true if last_happiness_survey_prompted_at.before?(Setting.load_safe("happiness_survey_period", 30).days.ago)

	false
end
```

En los settings:
```ruby
Setting.load_safe("happiness_survey_period")
```

- En Omega: 30
- En Alpha: 1

# Problema

El problema es que la definición de si se muestra el prompt está separada de guardar la encuesta.

El serializer `V2AccountSerializer` devuelve una bandera `should_prompt_for_happiness_survey?`:
```ruby
def should_prompt_for_happiness_survey?
	return false unless object.therapist?

	object.therapist.should_prompt_for_happiness_survey?
end
```

Este llama a la función del modelo que está antes.

Por su parte, el controlador solo se encarga de recibir parámetros y crear un nuevo resultado. Así que el cálculo de mostrar lo hace la app móvil según lo que le devuelva la bandera.

El bug podría estar por ese lado.