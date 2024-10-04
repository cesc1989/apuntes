# Error: class variable @@silencer of ActiveSupport::Logger is overtaken by Logger ✅

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

Este error está relacionado con este fix que hice antes para poder hacer el build en el CI [[Upgrade to Rails 7.0.x Notes ✅#undefined method silence for Logger]]

Tengo esto:
```ruby
# config/application.rb

Rails.logger.class.include ActiveSupport::LoggerSilence
```

Una vez comentado el servidor arranca debidamente.

## Pruebas que se rompen sin la configuración de LoggerSilence ✅

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

Funciona en local y en el CI.

# Error: partial con triple extensión no se encontraba ✅

Navegando en Customers -> Patients

Al partial: `app/views/images/_HiOutlineExternalLink.svg.html.erb` me tocó cambiarle las extensiones quitando la `svg`. Por alguna razón, no se encontraba con la triple extensión.

En la vista también tuve que actualizar la referencia:
```diff
# app/admin/customers/patients.rb

- controller.render_to_string(partial: "images/HiOutlineExternalLink.svg")
+ controller.render_to_string(partial: "images/HiOutlineExternalLink")
```

# Error: undefined method service_url for ActiveStorage attachment ✅

Navegando Files -> Incoming Faxes

Explota en esto:
```ruby
# app/models/concerns/attachment_methods.rb
if Rails.application.config.active_storage.service == :amazon
	public_send(attachment_association_name).service_url
```

Esto es una cosa por error de método depreciado:
```
DEPRECATION WARNING: service_url is deprecated and will be removed from Rails 7.0 (use url instead) (called from block in expose_attachment_url_for at /Users/francisco/projects/luna-project/backend/app/models/concerns/attachment_methods.rb:14)
```

Lo que no entiendo es porque no me salió nada al buscar en Internet.

**Arreglado**: en PR independiente.

# Error: undefined method gsub for nil:NilClass ❗️

Navegando Clinical -> Plans of Care

```
undefined method `gsub' for nil:NilClass

        str = str.gsub '\n', "\n"
                 ^^^^^

app/services/plans_of_care/plan_of_care_fax_pdf_service.rb:46:in `initialize'
app/models/plan_of_care.rb:312:in `new'
app/models/plan_of_care.rb:312:in `invalid_pdf_template?'
app/admin/clinical/plans_of_care.rb:244:in `block (3 levels) in <main>'
app/admin/clinical/plans_of_care.rb:236:in `block (2 levels) in <main>'
```

Esto parece solo ser en local porque me faltan variables de entorno de servicios de Google.

# Error: nil ID for rails url ✅

Navegando Clinical -> Protocol Escalations

```ruby
Showing _/Users/francisco/.gem/ruby/3.1.0/gems/activeadmin-2.14.0/app/views/active_admin/resource/index.html.arb_ where line **#3** raised:

No route matches {:action=>"show", :controller=>"admin/patients", :id=>nil}, missing required keys: [:id]

    column "Patient" do |escalation|
      patient = escalation.patient
      url = admin_patient_path(patient)
      link_to(patient.name, url, target: :_blank, rel: :noopener)
    end
    column "Physician" do |escalation|

app/admin/clinical/protocol_escalations.rb:41:in `block (3 levels) in <main>'
app/admin/clinical/protocol_escalations.rb:39:in `block (2 levels) in <main>'
```

¿por qué pasa en Rails 7? ¿Es esto de ActiveAdmin?

Algo que aún no descubro qué es cambió en Rails 7 y entonces ya no se puede acceder a la instancia de Patient del ProtocolEscalation. Tocó hacer esto:
```ruby
column "Patient" do |escalation|
	escalation.reload
	patient = escalation.patient

	url = admin_patient_path(patient)
	link_to(patient.name, url, target: :_blank, rel: :noopener)
end
```

Buscaré información para tratar de entender por qué y si es necesario dejar eso así.

Issues posiblemente relacionados:

- [has_one through not available on unpersisted instances](https://github.com/rails/rails/issues/33155)
- [has_one :through associations ignore preloaded records](https://github.com/rails/rails/issues/51817)
	- PR (opened) for this bug [[Fix #51817] Allow Preloaded has_one :through Associations to be Accessed Without Additional Database Queries](https://github.com/rails/rails/pull/52060)

## Fixed 🎉

Este problema se arregla en Rails 7.0.5. Es un problema con [asociaciones intermedias precargadas](https://github.com/rails/rails/issues/45822). Arreglado en este [pr](https://github.com/rails/rails/pull/46579).


# while_preventing_writes is only available on the connection_handler with legacy_connection_handling ✅

El Error:
```
NotImplementedError

while_preventing_writes is only available on the connection_handler with legacy_connection_handling
```

Se da en workers. Se arregla seteando en true `legacy_connection_handling` en application.rb. Ver [[004 - 👌🏽 AppBlend What Changes in Config Defaults#What is config.active_record.legacy_connection_handling?]]

# Timeout en Auto Charts ✅

Al ir a la página en alpha da time out y en local se queda cargando Accounts

![[auto.charts.timeout.rails7.png]]

Se está quedando pegado en esta parte:
```ruby
# app/models/concerns/profile.rb:9

included do
    belongs_to :account
    delegate :name, to: :account # AQUI
    delegate :first_name, to: :account
end
```

Pero no es nada raro. Solo está cargando datos y en Rails 6 y en Alpha también pasa.

> Lo más raro es que a pesar de que dio el timeout de la sesión de Luxe, se reinició la sesión y navegué otra página, siguió haciendo su ejecución de Accounts!?

La razón es que ==la conexión a la base de datos en el RDS es MUY lenta==. **Hice un dump local y todo carga muchísimo más rápido**.

# Timeout en Patient Duplicates ✅

Me pasó en alpha cuando probé. No me pasó en local??