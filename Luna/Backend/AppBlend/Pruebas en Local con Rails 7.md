# ✅ Error: class variable @@silencer of ActiveSupport::Logger is overtaken by Logger

Lancé el servidor Rails en local y cuando abrí la página admin dio error 500:
```bash
I, [2024-09-08T11:29:04.142050 #86103]  INFO -- : Completed 500 Internal Server Error in 9ms (ActiveRecord: 0.0ms | Allocations: 3142)


F, [2024-09-08T11:29:04.144265 #86103] FATAL -- :
RuntimeError (class variable @@silencer of ActiveSupport::Logger is overtaken by Logger):

activesupport (7.0.4) lib/active_support/logger_silence.rb:12:in `silencer'
activesupport (7.0.4) lib/active_support/logger_silence.rb:18:in `silence'
activerecord-session_store (2.1.0) lib/action_dispatch/session/active_record_store.rb:119:in `get_session_model'
activerecord-session_store (2.1.0) lib/action_dispatch/session/active_record_store.rb:147:in `find_session'
```

Parece ser un error en la gema `activerecord-session_store` según comentario en [este issue](https://github.com/rails/rails/issues/41053#issuecomment-758495500).

Se menciona causa y [fix que se mezcló](https://github.com/rails/activerecord-session_store/pull/159) a master pero no hay release. A pesar de que apunto a la rama master en el Gemfile sigue dando el error.

Este error está relacionado con este fix que hice antes para poder hacer el build en el CI [[Upgrade to Rails 7.0.4 - Notes#undefined method silence for Logger]]

Tengo esto:
```ruby
# config/application.rb

Rails.logger.class.include ActiveSupport::LoggerSilence
```

Una vez comentado el servidor arranca debidamente.

## Pruebas que se rompen sin la configuración de LoggerSilence

Estas son algunas de las pruebas:
```
pruebas ./spec/requests/graphql/queries/fields/node_care_plan_query_spec.rb:107

pruebas ./spec/requests/graphql/mutations/echo/echo_auto_chart_validate_subjective_five_spec.rb:38

pruebas ./spec/requests/graphql/queries/connections/appointments_query_spec.rb:1229

pruebas ./spec/requests/api/v2/therapist/auth_spec.rb

pruebas ./spec/requests/api/v2/patient/accounts_spec.rb:27
```

Una vez activo esa configuración para entorno pruebas, dejan de fallar:
```ruby
# config/environments/test.rb

Rails.logger.class.include ActiveSupport::LoggerSilence
```

# ✅ Error: partial con triple extensión no se encontraba

Al partial: `app/views/images/_HiOutlineExternalLink.svg.html.erb` me tocó cambiarle las extensiones quitando la `svg`. Por alguna razón, no se encontraba con la triple extensión.

En la vista también tuve que actualizar la referencia:
```diff
# app/admin/customers/patients.rb

- controller.render_to_string(partial: "images/HiOutlineExternalLink.svg")
+ controller.render_to_string(partial: "images/HiOutlineExternalLink")
```

# Error: undefined method service_url for ActiveStorage attachment

Explota en esto:
```ruby
# app/models/concerns/attachment_methods.rb
if Rails.application.config.active_storage.service == :amazon
	public_send(attachment_association_name).service_url
```