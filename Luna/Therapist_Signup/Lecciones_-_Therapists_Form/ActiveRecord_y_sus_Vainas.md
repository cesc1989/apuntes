# ActiveRecord y sus Vainas

# Validación de `ActiveRecord::RecordNotUnique`

Me encontraba trabajando en una tarea para controlar el error de unicidad de correo de terapeuta que se registra. Sin embargo, parece que hacerlo no es tan sencillo ni tan directo como se esperaba.

Lo primero es lo usual usando `rescue_from` en un controlador base. [En esta pregunta en StackOverflow](https://stackoverflow.com/questions/4982371/handling-unique-record-exceptions-in-a-controller) las respuestas dan varias opciones o formas de controlar este caso.

Algo que hay que tener en cuenta es que la validación de unicidad puede darse en dos partes en Rails.

Si se controla mediante la validación de unicidad de Rails

    validates :field, uniqueness: true

El error ocurre a nivel de aplicación, mucho antes de intentar guardar en la base de datos.

Por el otro lado, cuando la tabla tiene un índice único en el campo, cuando se intenta guardar un campo con duplicado, el error lo levanta la base de datos y es ahí donde se da la excepción `ActiveRecord::RecordNotUnique`.

Enlaces Relacionados
Leyendo e investigando encontré estos enlaces:

- Gema [*rescue unique constraint*](https://github.com/reverbdotcom/rescue-unique-constraint)
- Artículo [*Handling exceptions in Rails API applications*](https://driggl.com/blog/a/handling-exceptions-in-rails-applications)
- Artículo [*RecordNotUnique - Expect the unexpected*](http://benjit.com/rails/activerecord/2015/04/03/uniqueness-constraint-the-expected-exception/)


# Migración de Tipo de Campo de `string` a `date`

Tenía que hacer una migración del campo `date_of_birth` del modelo Therapist. Inicialmente estaba creado como tipo `string` pero la migración requería moverlo a un tipo `date`. Para esos casos, la migración tiene que ser de este estilo:


    change_column :users, :date_of_birth, 'date USING CAST(date_of_birth AS date)'

Sin embargo, como había algunos registros donde este valor estaba vació(“”) el siguiente error aparecía:


    PG::InvalidDatetimeFormat: ERROR:  invalid input syntax for type date: ""

Para solucionarlo hay dos formas. Una es en una tarea limpiar todos los valores vacíos y volverlos `null` y la otra es [correr la migración de esta forma](https://stackoverflow.com/a/42394799/1407371):


    change_column :users, :date_of_birth, 'date USING CAST(case when date_of_birth = '' then null else date_of_birth end AS date)'


# Error por campo de tipo integer que llegó a su límite

El campo hubspot_id de la tabla therapists era de tipo Integer.

Esos tienen un límite de 4 bites el cual fue alcanzado en Therapists alpha al intentar guardar un hubspot_id:

    5162756985 is out of range for ActiveModel::Type::Integer with limit 4 bytes

La solución es cambiar el tipo de campo a bigint

    change_column :therapists, :hubspot_is, :bigint

Visto en Stack Overflow [https://stackoverflow.com/a/56190726/1407371](https://stackoverflow.com/a/56190726/1407371)

