# Lecciones Rails - Luna Patients

Lecciones aprendidas mientras trabajaba en el desarrollo de los formularios para pacientes.

# Explicación de Métodos Simples y Poderosos

Al usar `#find_or_initialize_by` en varias ocasiones para la implementación de los borradores de los formularios, me dio curiosidad sobre cómo funciona. [Es bastante simple su lógica](https://api.rubyonrails.org/classes/ActiveRecord/Relation.html#method-i-find_or_initialize_by):

```ruby
# File activerecord/lib/active_record/relation.rb, line 176
def find_or_initialize_by(attributes, &block)
  find_by(attributes) || new(attributes, &block)
end
```

También es una demostración de **lo poderosos que son los bloques** y que **debería aprender a usarlos más**.

Otro dato curioso fue [la implementación de](https://api.rubyonrails.org/classes/ActiveRecord/Relation.html#method-i-size) `#size`

```ruby
# File activerecord/lib/active_record/relation.rb, line 210
def size
  loaded? ? @records.length : count(:all)
end
```

Que significa que si la colección ya está cargada en memoria, utiliza el método `#length`, lo que significa que se a nivel de Ruby o el framework, mientras que en caso contrario, se va a la base de datos al ejecutar un `#count(:all)`.

# Pasar Valor a Serializer desde Controlador

Se describe en [esta página de la documentación](https://github.com/rails-api/active_model_serializers/blob/v0.10.7/docs/howto/passing_arbitrary_options.md) como lograr tal hazaña. En realidad es bastante sencilla:

```ruby
# Controlador
def index
  previous_intake = @form.patient.previous_intake

  if previous_intake
	render json: previous_intake,
		   current_form_type: @form.type_name,
		   serializer: PreviousIntakeFormSerializer,
		   status: :ok
  else
	head :no_content
  end
end

# Serializer
def answers_attributes
  return nil if instance_options[:current_form_type] != object.type_name

  # ...
end
```

# Incluir o excluir atributos al usar `as_json`

Para convertir los atributos de una instancia de un modelo `ActiveRecord` a un `Hash`, además del método `attributes` [se puede usar](https://apidock.com/rails/ActiveModel/Serializers/JSON/as_json) `[as_json](https://apidock.com/rails/ActiveModel/Serializers/JSON/as_json)`.

Con `as_json` se pueden usar las opciones *only* y *except* para indicar qué atributos deberán estar en el resultado.

Ejemplo:

    let(:aggravating_activities_attributes) do
      previous_intake.aggravating_activities.map { |aggr| aggr.as_json(except: [:id]) }
    end

En el ejemplo, todos los atributos de cada *Aggravating Activity* estarán en el Hash exceptuando el atributo **id**.

Con `#attributes` también se puede lograr parte de lo que hace `as_json` siendo la mayor diferencia que [con el primer método](https://apidock.com/rails/ActiveRecord/AttributeMethods/attributes) solo se retorna el hash sin ninguna posibilidad de excluir/indicar los campos que se necesiten.

# Dejar de Usar RVM Ruby como por defecto

Normalmente, al instalar Ruby con RVM, se configura por defecto esa versión. Para poder dejar de usar dicho Ruby en ese modo, [se puede usar el comando](https://rvm.io/rubies/default) `rvm reset` para que se use la del sistema.

En cualquier momento se puede ejecutar `rvm use 2.5.3` para usar la versión instalada.

# Cambiar versión por defecto de bundler

Al parecer, al instalar bundler también se configura una versión por defecto. [Para cambiarla, se puede intentar con](https://stackoverflow.com/questions/54761120/how-to-change-bundler-default-version):


    $ bundle config default 2.0.1

Sin embargo, al listar las versiones de bundler, este comando parece no surtir efecto:


    $ gem list bundler
    
    *** LOCAL GEMS ***
    
    bundler (2.0.1, default: 1.17.3)

Pero en el archivo `~/.bundle/config` se puede ver la configuración efectuada.


    $ cat /Users/iwbackend/.bundle/config
    ---
    BUNDLE_DEFAULT: "2.0.1"

Ver `bundle-config --help` para más información de este comando.

# Prevenir consultas extra usando `ActiveRecord::Relation#load`

Cuando se quiere verificar si una asociación tiene registros, normalmente se usaría el método `#empty?`.

En este caso:

```ruby
def medications
  return [] if object.medications.empty?

  meds = object.medications.map do |med|
	next if med.name.blank?
	med.as_json(only: [:id, :name, :dosage])
  end

  meds.compact!
end
```

Se ejecutaran dos consultas, como lo muestra los registros:

    Started GET "/patients/99985/intake_forms/61" for 127.0.0.1 at 2019-06-18 09:54:12 -0500
    Processing by Api::V1::IntakeFormsController#show as JSON
    ...
    [active_model_serializers]   Medication Exists (0.6ms)  SELECT  1 AS one FROM "medications" WHERE "medications"."intake_form_id" = $1 LIMIT $2  [["intake_form_id", 61], ["LIMIT", 1]]
    [active_model_serializers]   Medication Load (0.7ms)  SELECT "medications".* FROM "medications" WHERE "medications"."intake_form_id" = $1  [["intake_form_id", 61]]
    ...
    [active_model_serializers] Rendered IntakeFormSerializer with ActiveModelSerializers::Adapter::Json (167.84ms)
    Completed 200 OK in 413ms (Views: 152.4ms | ActiveRecord: 81.6ms)

La primera consulta es para saber si el intake form tiene *medications* y la segunda es la consulta que trae todos los registros de la relación.

Si se usara el método `#load` en la primera consulta, solo sería una en total:

```ruby
 def medications
  return [] if object.medications.load.empty?

  meds = object.medications.map do |med|
	next if med.name.blank?
	med.as_json(only: [:id, :name, :dosage])
  end

  meds.compact!
end
```

Resultado:

    Started GET "/patients/99985/intake_forms/61" for 127.0.0.1 at 2019-06-18 09:57:09 -0500
    Processing by Api::V1::IntakeFormsController#show as JSON
    ...
    [active_model_serializers]   Medication Load (0.5ms)  SELECT "medications".* FROM "medications" WHERE "medications"."intake_form_id" = $1  [["intake_form_id", 61]]
    [active_model_serializers]   Surgery Load (0.5ms)  SELECT "surgeries".* FROM "surgeries" WHERE "surgeries"."intake_form_id" = $1  [["intake_form_id", 61]]
    ...
    [active_model_serializers] Rendered IntakeFormSerializer with ActiveModelSerializers::Adapter::Json (202.52ms)
    Completed 200 OK in 507ms (Views: 196.0ms | ActiveRecord: 65.9ms)

Una sola consulta en total.

El uso de `load` lo aconseja Nate Berkopec en su artículo [*3 ActiveRecord Mistakes That Slow Down Rails Apps: Count, Where and Present*](https://www.speedshop.co/2019/01/10/three-activerecord-mistakes.html)*.*

> load just causes all of the records described by @messages to load immediately, rather than lazily. It returns the ActiveRecord::Relation, not the records. So, when size is called, the records are loaded? and a query is avoided. Voilà.

Definición del método en el código fuente [ActiveRecord::Relation#load](https://apidock.com/rails/ActiveRecord/Relation/load)

```ruby
def load(&block)
  exec_queries(&block) unless loaded?

  self
end
```

# Conectar la API corriendo localmente con la App frontend corriendo localmente entre máquinas en la misma red

Esto se logra usando el archivo `/etc/hosts` con el cual se pueden crear aliases para direcciones IP o dominios.

La idea es poder conectar el API corriendo en el computador del backend con la App corriendo en el computador del frontend.

Se necesita saber la IP del computador del backend con el comando `ifconfig`:

    $ ifconfig | grep 10
    gif0: flags=8010<POINTOPOINT,MULTICAST> mtu 1280
    inet 10.1.184.194 netmask 0xffffff00 broadcast 10.1.184.255
    maxage 0 holdcnt 0 proto stp maxaddr 100 timeout 1200
            ifmaxaddr 0 port 10 priority 0 path cost 0

La IP aparece al lado de *inet.*

Ya con esto, en el computador del frontend se modifica el archivo `/etc/hosts` de la siguiente forma:

    $ sudo nano /etc/hosts
    
    ##
    # Host Database
    #
    # localhost is used to configure the loopback interface
    # when the system is booting.  Do not change this entry.
    ##
    127.0.0.1       localhost
    255.255.255.255 broadcasthost
    ::1             localhost
    
    10.1.184.194 api.patients.dev

Para el caso de proyectos en Rails que tienen el *namespace* API y tienen una restricción para solo recibir peticiones desde un subdominio **api** hay que indicarlo al crear el alias.

# String#start_with? acepta un array

Docs → https://apidock.com/ruby/String/start_with%3F

Código:

```ruby
def therapist_path?
  request.fullpath.start_with?('/v2/therapist', '/v3/therapist')
end
```

