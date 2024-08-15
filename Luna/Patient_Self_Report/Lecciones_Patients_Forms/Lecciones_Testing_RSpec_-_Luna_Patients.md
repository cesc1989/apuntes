# Lecciones Testing RSpec - Luna Patients

## Pruebas donde se compara fechas dadas con la clase `Time`

Resulta que la clase `Time` en Ruby mantiene una precisión mucho mayor que la base de datos:


> When the value is read back from the database, it’s only preserved to microsecond precision, while the in-memory representation is precise to nanoseconds.
> 
> [Stack Overflow](https://stackoverflow.com/a/20403290/1407371)

Por eso, en los tests de configurar el TTL de los borradores del onboarding y el progress, cuando hacía:


    persist_until = Time.zone.at(form.persist_draft_until)
    expect(persist_until).to eq expire_next_week

Las pruebas no pasaban por diferencia de nanosegundos


    Failure/Error: expect(persist_until).to eq expire_next_week
         
           expected: 2019-05-28 14:45:42.479428000 +0000
                got: 2019-05-28 14:45:42.000000000 +0000
         
           (compared using ==)

Para que los tests pasen hay que enviar el método `to_s` o `to_i` a la instancia de `Time`


    persist_until = Time.zone.at(form.persist_draft_until)
    expect(persist_until.to_i).to eq expire_next_week.to_i


## Cómo Usar FactoryBot en la consola de Rails

Opción uno


    $ rails console test

Opción dos, más idónea por lo que menciona [la respuesta en Stack Overflow](https://stackoverflow.com/a/23580836/1407371)


    $ rails console test --sandbox


## Cómo Saltarse las Validaciones al Crear Factories

*O cómo hacer skip de validación en rspec. (lo pongo así para poder encontrarlo al buscar).*

No hay una forma directa usando la sintaxis de FactoryBot pero [sí hay formas de hacerlo](https://stackoverflow.com/questions/7652956/bypass-rails-validations-when-creating-factorygirl-objects).

**Forma A**
En esta forma se logra el objetivo pero lo retornado, si se necesita, solo es una instancia de `TrueClass` y no una instancia del modelo:

    FactoryGirl.build(:model_a).save(validate: false)

**Forma B**
Si se necesita la instancia del modelo:

    model_a = FactoryGirl.build(:model_a)
    model_a.save(validate: false)


## Generar una Lista de Atributos de un Factory

Normalmente, si se quiere generar una lista de *factories* se usa el método `FactoryBot.create_list(:factory_name, amount)`. El resultado de dicha invocación es un arreglo de registros de la fábrica usada.

Cuando se están probando APIs, en una petición se envía un objeto JSON. En términos de RSPec, se envía un hash con los atributos del registro a generar.

FactoryBot deja generar una lista de objetos JSON/hashes para enviar en una petición donde se puedan guardar varios registros a la vez, ejemplo, una relación anidada(*nested*).

En [la documentación](https://github.com/thoughtbot/factory_bot/blob/master/GETTING_STARTED.md#building-or-creating-multiple-records):

    # If you need multiple attribute hashes, attributes_for_list will generate them:
    users_attrs = attributes_for_list(:user, 25) # array of attribute hashes


## Cómo Mockear Variables de Entorno

Resulta que en el ambiente de pruebas en Circle CI, las variables de entorno de AWS no están:

    AWS_ACCESS_KEY_ID="KEY"
    AWS_SECRET_ACCESS_KEY="KEY"
    AWS_S3_BUCKET_NAME="lcpatients"
    AWS_REGION="us-east-1"

Entonces, al correr las pruebas, cuando carga la aplicación da error indicando que no hay nada en dichas variables:

    Failure/Error:
            @s3_resource = Aws::S3::Resource.new(
              region: ENV['AWS_REGION'],
              access_key_id: ENV['AWS_ACCESS_KEY_ID'],
              secret_access_key: ENV['AWS_SECRET_ACCESS_KEY']
            )
          
          Aws::Errors::MissingRegionError:
            missing region; use :region option or export region name to ENV['AWS_REGION']

A pesar de que estoy mockeando llamados a métodos específicos del SDK, la configuración no está mockeada. Para solucionar o hay que poner las variables de entorno para que Docker las lea o buscar la forma de mockearlas cuando se ejecuten las pruebas.

**Mockeando o Apuñalando Variables de Entorno en RSpec**
Encontré varias formas.

En este [artículo de Thoughtbot](https://thoughtbot.com/blog/testing-and-environment-variables), encontré esta forma:

    allow(ENV).to receive(:[]).with('AWS_REGION').and_return('us-east-1')

Pero daba error

    expected: ("AWS_REGION")
                  got: ("MARKETPLACE_API_TOKEN")
            Please stub a default value first if message might be received with other args as well. 

Lo que entiendo es que a `ENV` solo le estaba dando una sola variable de entorno y todo lo demás explotaba por eso.

Entonces intenté:

    allow(ENV).to receive(:[]).with('AWS_REGION').and_return('us-east-1')
    allow(ENV).to receive(:[]).with('MARKETPLACE_API_TOKEN').and_return('ATOKEN')

Pero seguía dando errores.

Luego, probé con esta otra forma que encontré en [Stack Overflow](https://stackoverflow.com/questions/9611276/what-is-the-best-way-to-write-specs-for-code-that-depends-on-environment-variabl) y sí dio mejores resultados.

    stub_const('ENV', {'AWS_REGION' => 'us-east-1'})

Y como de esa forma sí sirvió, procedí a crear un objeto general que tuviera las variables y valores falsos de AWS y luego usar `stub_const` para las veces que fueran necesarios:

    RSpec.describe 'OnboardingForms Requests', type: :request do
      let(:aws_options) do
        {
          'AWS_REGION' => 'us-east-1',
          'AWS_ACCESS_KEY_ID' => 'KEY_ID',
          'AWS_SECRET_ACCESS_KEY' => 'SECRET_KEY',
          'AWS_S3_BUCKET_NAME' => 'lcpatient'
        }
      end
      # (...)
      before do
        stub_const('ENV', ENV.to_hash.merge(aws_options))
      end
    end

De esa forma tenía todas las variables que ya estaban siendo leídas por `ENV` y adicional mockeaba las que me interesan de AWS.

**Enlaces Relacionados**

- [Testing and Environment Variables](https://thoughtbot.com/blog/testing-and-environment-variables).
- [Is there a way to stub Rails credentials key?](https://github.com/rspec/rspec-rails/issues/2099)
- [Pregunta en Stack Overflow](https://stackoverflow.com/questions/23146353/rspec-3-0-how-to-mock-a-method-replacing-the-parameter-but-with-no-return-value)
- [GitLab Testing Best Practices](https://docs.gitlab.com/ee/development/testing_guide/best_practices.html)
- [What is the best way to write specs for code that depends on environment variables?](https://stackoverflow.com/questions/9611276/what-is-the-best-way-to-write-specs-for-code-that-depends-on-environment-variabl)
- [stub_env gem](https://github.com/ljkbennett/stub_env)


## Cómo generar un uuid en entorno pruebas

Cuando el campo en la base de datos no es un `uuid` nativo, ¿cómo generarlos en las pruebas? Pues, con `SecureRandom.uuid`


    care_plan_id { SecureRandom.uuid }

😄 Visto en [este issue](https://github.com/thoughtbot/factory_bot/issues/576#issuecomment-29735771).


## Cómo emular la carga de un archivo

Usando `fixture_file_upload`. Ver [docs](https://apidock.com/rails/ActionDispatch/TestProcess/FixtureFile/fixture_file_upload).

    fixture_file_upload(Tempfile.new("signed_terms.pdf"), "application/pdf")


## Usando Tempfile

[Docs sobre Tempfile](https://rubyapi.org/3.2/o/tempfile).


    Tempfile.new("signed_terms.pdf")
    
    Tempfile.new(["signed_terms", ".pdf"])

