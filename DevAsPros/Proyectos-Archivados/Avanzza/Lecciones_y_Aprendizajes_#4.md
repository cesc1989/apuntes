# Lecciones y Aprendizajes #4

- Para separar las rutas en archivos diferentes para mayor comodidad: [https://mattboldt.com/separate-rails-route-files/](https://mattboldt.com/separate-rails-route-files/)
- Simple Form associations input explained https://github.com/plataformatec/simple_form#associations

En los campos tipo *number* el límite debería mejor contralarse por JS ya que los atributos *min* y *max* no impiden ingresar un mayor número de campos:

- https://stackoverflow.com/questions/8354975/how-can-i-limit-possible-inputs-in-a-html5-number-element
- Acá una respuesta sugiere usar input type tel https://stackoverflow.com/questions/22086823/limit-number-of-characters-in-input-type-number

Según este comentario en in Issue, la validación de *length* solo es para strings, no para integers.

- https://github.com/thoughtbot/shoulda-matchers/issues/970#issuecomment-292345134

**Lista de selección con meses del año**
Se puede lograr de diferentes formas:

- Usando el helper `[select_month](https://apidock.com/rails/ActionView/Helpers/DateHelper/select_month)`. Este no pude hacerlo funcionar con simple form. Visto en [SO](https://stackoverflow.com/questions/13001904/rails-drop-down-select-month-year)
- Otra forma es usando las constantes de la clase Date: `<%= f.select :end_date, Date::MONTHNAMES[1..12] %>`


## Reiniciar dyno en Heroku cuando no se refleje cambio de migración

Visto en [Stack Overflow](https://stackoverflow.com/questions/11579227/heroku-undefined-method-name-for-user).

    # para reiniciar la aplicación.
    heroku restart


## Correr migraciones en despliegue usando *Release phase*

[Heroku Dev Center](https://devcenter.heroku.com/articles/release-phase).
Poner en el Procfile

    release: bundle exec rails db:migrate
    web: rails server
    worker: bundle exec sidekiq


## Cómo traducir enums con i18n

Primero que nada, tener configurado el idioma en `config/application.rb`

    config.i18n.default_locale = :es

Para un enum:

    class Pqr < ApplicationRecord
      enum status: { unassigned: 0, assigned: 1, closed: 2 }
    end

Se configura la traducción así:

    # config/locales/es.yml
    es:
      pqr_status:
        unassigned: 'Sin asignar'
        assigned: 'Asignado'
        closed: 'Cerrado'
    

Y se usan en la vista:

    <%= I18n.t(:"pqr_status.#{pqr.status}") %>

Visto en: [Stack Overflow](https://stackoverflow.com/questions/22827270/how-to-use-i18n-with-rails-4-enums). [Otra](https://stackoverflow.com/questions/43116134/rails-i18n-how-to-translate-enum-of-a-model) forma.


## ActiveStorage

No se recomiendo agregar campos a las tablas de ActiveStorage. Al contrario sería crear una tabla la cual tendría los campos necesarios y ese modelo luego tendría la relación con ActiveStorage.
Ver [Stack Overflow](https://stackoverflow.com/questions/49510195/rails-5-2-active-storage-add-custom-attributes)

En este otro se indica que se pueden agregar campos y diversas formas para que las modificaciones estén disponibles en tiempo de ejecución: [Stack Overflow](https://stackoverflow.com/questions/50770660/extending-activestorageattachment-adding-custom-fields)

