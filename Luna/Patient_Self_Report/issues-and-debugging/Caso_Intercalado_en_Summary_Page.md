# Caso Intercalado en Summary Page
Para el 1er form que tengo en mi local, el intercalado luce así:

![](https://paper-attachments.dropbox.com/s_7C1D7A51C7FC94E4C49571F03424FD3EE4323963B99589DAC4A855B2936FE2BE_1612366428329_image.png)


Este es el JSON:

    "forms": [
        {
            "id": 19,
            "created_at": "2020-06-03",
            "completed": false,
            "completed_at": null,
        },
        {
            "id": 8,
            "created_at": "2020-05-06",
            "completed": true,
            "completed_at": "2020-10-19",
        },
        {
            "id": 18,
            "created_at": "2020-06-03",
            "completed": true,
            "completed_at": "2020-11-27",
        },
        {
            "id": 1
        }
    ],

Cuando se usa el ordenamiento así:

    order('completed_at DESC NULLS LAST')

Luce de esta manera:

![](https://paper-attachments.dropbox.com/s_7C1D7A51C7FC94E4C49571F03424FD3EE4323963B99589DAC4A855B2936FE2BE_1612366572848_image.png)


El cual ya luce más intercalado pero aún falla. Este es su JSON:

    "forms": [
        {
            "id": 8,
            "created_at": "2020-05-06",
            "completed": true,
            "completed_at": "2020-10-19",
        },
        {
            "id": 18,
            "created_at": "2020-06-03",
            "completed": true,
            "completed_at": "2020-11-27",
        },
        {
            "id": 19,
            "created_at": "2020-06-03",
            "completed": false,
            "completed_at": null,
        },
        {
            "id": 1,
        }
    ],


## La Causa

El error se debe a este cambio introducido por Steve Sims en `Form.rb`.

    has_many :ongoings,
    +        -> { order(created_at: :asc) },
            class_name: 'Form',
            foreign_key: 'onboarding_id',
            dependent: :destroy

La lambda para darle un orden por defecto a los progress forms del Intake Form. Cuando quito eso, todo vuelve a la normalidad.

¿Cómo solucionarlo? Usando una asociación diferente para los Progress Forms en el Summary Page o lo que hizo Steve.

Solucionado con:

    has_many :unordered_ongoings,
             class_name: 'Form',
             foreign_key: 'onboarding_id'

