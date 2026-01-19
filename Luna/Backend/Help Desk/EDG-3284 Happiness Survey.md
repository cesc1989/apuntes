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

