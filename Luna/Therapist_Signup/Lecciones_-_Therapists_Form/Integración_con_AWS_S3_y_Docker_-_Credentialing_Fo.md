# Integración con AWS S3 y Docker - Credentialing Form

Para la integración con AWS S3 **decidí no usar** `ActiveStorage` ni nada similar ya que solo necesito subir el archivo, obtener su URL pública y actualizar el campo en el registro.

Usar `ActiveStorage` implica contar con una tabla solo para guardar los *blobs* y además tiene como desventaja que todo queda almacenado en el *BUCKET* definido. No hay forma de dar una estructura a los archivos según el Terapeuta que los carga.

Este [issue de Rails](https://github.com/rails/rails/issues/32790) trata un poco al respecto y no hay forma ni planes de soportar algo así.

Al final, la integración la hice usando la gema *aws-sdk-s3* y un [PORO](http://blog.jayfields.com/2007/10/ruby-poro.html) que usa la clase `AWS::S3::Resources` para tener un cliente con el cual interactuar con el servicio.

La forma en que se subirán los archivos será con el [método](https://docs.aws.amazon.com/sdkforruby/api/Aws/S3/Object.html#put-instance_method) `[#put](https://docs.aws.amazon.com/sdkforruby/api/Aws/S3/Object.html#put-instance_method)` de la [clase](https://docs.aws.amazon.com/sdkforruby/api/Aws/S3/Object.html) `[Object](https://docs.aws.amazon.com/sdkforruby/api/Aws/S3/Object.html)`. Inicialmente lo iba a hacer con el [método](https://docs.aws.amazon.com/sdkforruby/api/Aws/S3/Client.html#put_object-instance_method) `[#put_object](https://docs.aws.amazon.com/sdkforruby/api/Aws/S3/Client.html#put_object-instance_method)` de la clase `Client` pero las instancias de esta no cuentan con una forma para obtener la URL pública del recurso cargado. Con `Object` se puede mediante `#public_url`.

# Los Permisos en IAM

Todo quedó así:

- Creé un usuario con acceso programático(por API) y conservé el *access key* y el *secret key*
- Lo añadí a un grupo que tiene la política siguiente:
```json
{
  "Version": "2012-10-17",
  "Statement": [
	{
	  "Effect": "Allow",
	  "Action": "s3:*",
	  "Resource": "*"
	}
  ]
}
```

Sin embargo, cuando intentaba cargar un objeto de la siguiente forma:
```ruby
obj = @s3_client.bucket(@bucket).object("credentialing/no-#{file.original_filename}")
obj.put(body: file, acl: 'public-read')
```

Se levantaba la excepción:

    Aws::S3::Errors::AccessDenied - Access Denied

Porque en el *bucket* las configuraciones de acceso público estaban desactivadas:

- *Block all public access*: On.

Si bien intenté con lo que sugirió en [esta respuesta en Stack Overflow](https://stackoverflow.com/a/32339576/1407371), fue insuficiente y finalmente las indicaciones de [esta otra respuesta](https://stackoverflow.com/a/36272287/1407371) fueron las que me permitieron cargar el archivo de la forma implementada.

Para impedir que la excepción apareciera tocó desactivar las dos de nombre:

1. *Block public access to buckets and objects granted through new access control lists (ACLs)*
2. *Block public access to buckets and objects granted through any access control lists (ACLs)*

O como se ve en la imagen:
![[s3.bucket.access.png]]

# Clases y Métodos de Ruby/Rails Descubiertos en la Integración

- [Para enviar](https://api.rubyonrails.org/classes/ActionController/Parameters.html#method-i-to_h) `#map` a una instancia de `ActionController::Parameters` hay que enviar el método  `[#to_h]`
    - `#to_h` retorna una instancia de `HashWithIndifferentAccess` mientras que `#to_hash` retorna una instancia de `Hash`
- Para saber la [extensión de un archivo](https://ruby-doc.org/core-2.5.3/File.html#method-c-extname) (recibe un *string*) se puede usar `File.extname`
- [FileUtils::copy](https://ruby-doc.org/stdlib-2.5.1/libdoc/fileutils/rdoc/FileUtils.html#method-c-copy), para copiar archivos
- [File::open](https://ruby-doc.org/core-2.5.3/File.html#method-c-open), para abrir archivos


## Clase UploadedFile de ActionDispatch::Http

[Docs](https://api.rubyonrails.org/classes/ActionDispatch/Http/UploadedFile.html).

> Models uploaded files.
> 
> The actual file is accessible via the `tempfile` accessor, though some of its interface is available directly for convenience.
> 
> Uploaded files are temporary files whose lifespan is one request. When the object is finalized Ruby unlinks the file, so there is no need to clean them with a separate maintenance task.

Esta clase es clave porque en la carga de archivos en Rails uno se puede topar con el uso de `#original_filename` pero ese método no lo tiene ni la clase [Tempfile](https://rubyapi.org/3.2/o/tempfile) ni [File](https://ruby-doc.org/core-3.1.0/File.html).

Ese método `#original_filename` lo que hace es usar el método `.basename` de la clase File

    [RW]        original_filename        The basename of the file in the client.

[Docs](https://ruby-doc.org/core-3.1.0/File.html#method-c-basename) de `File.basename`:

    File.basename("/home/gumby/work/ruby.rb")          #=> "ruby.rb"
    File.basename("/home/gumby/work/ruby.rb", ".rb")   #=> "ruby"
    File.basename("/home/gumby/work/ruby.rb", ".*")    #=> "ruby"

Cuando se pasa el sufijo (2do parámetro), se elimina del resultado final si lo hay.

Ejemplo:

    tmp_file = Tempfile.new(["signed_terms", ".pdf"], "./storage")
    ap File.basename(tmp_file.path)
    # "signed_terms20230721-49961-60ih0u.pdf"

# Enviar Archivos en Segundo Plano

Can't only use `ActiveJob` because in production, it needs a *queueing backend*.

> that is to say you need to decide for a 3rd-party queuing library that Rails should use

By default `ActiveJob` provides an async adapter but is only suitable for dev/test env:

> This is the default queue adapter. It's well-suited for dev/test since it doesn't need an external infrastructure, but it's a poor fit for production since it drops pending jobs on restart.
> [ActiveJob::QueueAdapters::AsyncAdapter](https://api.rubyonrails.org/v5.2.3/classes/ActiveJob/QueueAdapters/AsyncAdapter.html)

Iba a elegir **Sidekiq** pero Ryan hizo la aclaración que ellos ya tienen todo listo para usar **Resque**, por lo cual me decanté por **Resque** luego de ver que también se sencilla como Sidekiq.

# Implementación Propia Enfrenta Límites cuando se Intenta Mandar Archivos en Segundo Plano

Al intentar acomodar la implementación para funcionar enviando los archivos en segundo plano, me topo con varios problemas que tienen un mismo origen:

**A un** ***worker*** **no debes enviarle una instancia de una clase porque la va a serializar a un objeto JSON**.

Como la idea de segundo plano es guardar la información en un almacenamiento como Redis, en este no se puede guardar una instancia sino que hay que guardar un JSON.

Con eso en mente, cuando hice el Job de la siguiente forma:

    class S3UploadsJob
      @queue = :s3_uploads
    
      def self.perform(params, receiver_id, receiver_class)
        S3Uploads.new(
          uploadables: Uploadables.new(params).build,
          receiver: receiver_class.constantize.find(receiver_id)
        ).process
      end
    end

No funcionará porque en `params` viaja una instancia de `ActionDispatch::Http::UploadedFile` y al llegar al *job*, se serializa para que se pueda guardar en Redis.

**Lo que no funcionó fue**
Un intento que hice fue pasar `Uploadables.new(params).build` como parámetro en vez de `params` y funciona pero el archivo sube vacío. Esto es lo que tengo que descubrir por qué no funciona ya que intenté instanciando `ActionDispatch::Http::UploadedFile` e incluso `Tempfile` pero el archivo sigue subiendo vacío.

**Lo que sí funciono fue**
En vez de tratar de crear instancias de `Tempfile` en base al archivo en la carpeta temporal que se obtiene mediante `ActionDispatch::Http::UploadedFile`, lo que funcionó fue copiar el archivo de una carpeta a otra.

¿Por qué copiar? Resulta que archivos de tipo imagen u otros que no sean texto plano, su contenido no se puede copiar con hacer algo como


    Tempfile.new(['archivo', '.extension']).write(archivo.pdf)

Ya que su contenido está dado en un *blob*. Por eso es que `ActiveStorage` usa una tabla ya que en ella es donde almacena los *blobs* de los archivos que cargan.

Entonces teniendo en cuenta eso, lo que hice es lo siguiente en la línea 7

    class Uploadable
        # (...)
        def initialize(name, file)
          @name = name
          @original_filename = file.original_filename
    
          FileUtils.cp(file.tempfile, "#{TEMP_PATH}/#{@original_filename}")
        end
      end

Copio el archivo de la carpeta temporal inicial a otra carpeta. Esto es necesario porque una vez acabe el proceso de la petición, el `Tempfile` se borrará. Al crear una copia emulo el comportamiento como si tuviera una tabla al estilo `ActiveStorage`.

# Cerrar o no un Archivo Abierto Mediante  `File.open`

La duda nace por esto:


    # app/uploaders/s3_upload.rb
    class S3Uploader
      # (...)
    
      def initialize(uploadable, receiver)
        # (...)
        @file = prepare_file
      end
    
      def upload
        @s3_object.put(body: @file)
        @s3_object
      rescue StandardError => e
        Rails.logger.debug("#{e.class.name} - #{e.message}")
      end
    
      # (...)
    
      private
    
      def prepare_file
        File.open("#{TEMP_PATH}/#{@uploadable.original_filename}", FILE_MODE)
      end
    
      # (...)
    end

Claramente vemos que no estoy cerrando el archivo haciendo algún llamado a `@file.close` o al usar la sintaxis de bloque de `File.open`.

Cómo no comprendo bien las implicaciones de dejar un archivo abierto me di a la tarea de averiguar. Esto es lo que encontré.

**En Ruby Bastards Book,** [**capítulo IO**](http://ruby.bastardsbook.com/chapters/io/)

> What happens if you don't close a file? **Nothing too bad, usually**. But try writing a large amount of data to a file and have the program finish immediately after the write operation.

Entonces, ¿por qué cerrar los archivos?

> Similar to flushing in the real-world, doing a "flush" is good practice in programming because it frees up memory for the rest of your program and (ideally) ensures that that file is available for other processes to access.

[**En Stack Overflow**](https://stackoverflow.com/questions/4795447/rubys-file-open-and-the-need-for-f-close)
El OP(*original poster*) pregunta por qué es necesario cerrar los archivos cuando en la documentación del [método](https://ruby-doc.org/core-2.6.4/IO.html#method-i-close) `[close](https://ruby-doc.org/core-2.6.4/IO.html#method-i-close)` de la clase IO dice:

> I/O streams are automatically closed when they are claimed by the garbage collector.

A lo que responden:

> I assume you can easily run out of available file descriptors before the garbage collector cleans up. in this case, you might want to use close them yourself. "claimed by the garbage collector." means that the GC acts at some point in the future. and it's expensive. a lot of reasons for explicitly closing files.

La [respuesta elegida](https://stackoverflow.com/a/4795782/1407371):

> You simply cannot predict when the garbage collector will run. You cannot even predict if it will run at all: if you never run out of memory, the garbage collector will never run, therefore the finalizer will never run, therefore the file will never be closed.

**¿Qué es un** ***file descriptor*****?**
Veo que mencionan en varias ocasiones la importancia de cerrar el archivo para liberar su *file descriptor,* pero ¿qué son?

Según [Computer Hope](https://www.computerhope.com/jargon/f/file-descriptor.htm):

> A **file descriptor** is a number that uniquely identifies an open file in a computer's operating system. It describes a data resource, and how that resource may be accessed.
> 
> When a process makes a successful request to open a file, the kernel returns a file descriptor which points to an entry in the kernel's **global file table**. The file table entry contains information such as the [inode](https://www.computerhope.com/jargon/i/inode.htm) of the file, byte [offset](https://www.computerhope.com/jargon/o/offset.htm), and the access restrictions for that data stream(read-only, write-only, etc.).
![File descriptor diagram](https://www.computerhope.com/jargon/f/file-descriptor.jpg)


En [Stack Overflow](https://stackoverflow.com/questions/10922045/where-does-ruby-keep-track-of-its-open-file-descriptors) encuentro que Ruby mantiene información de los descriptores de archivos en el `ObjectSpace`.

# No se puede esta Integración Asíncrona con contenedores Docker

En local todo funciona bien porque tanto la aplicación Rails como los trabajadores de Resque corren en la misma máquina. La cosa cambia al desplegar a *staging.* En este entorno tanto Resque como Rails corren en contenedores independientes y no comparten un mismo sistema de archivos. Por ejemplo, [este issue](https://github.com/dokku/dokku/issues/2853).

Esto significa que la operación:

    FileUtils.cp(file.tempfile, "#{TEMP_PATH}/#{@original_filename}")

Funciona en el contenedor de Rails pero el archivo no existirá en el contenedor de Resque. Y tiene todo el sentido. Como esto no funciona quedaba la alternativa de usar una gema para gestionar la carga de archivos o dejar el proceso de manera síncrona.

Preferí dejarlo de forma síncrona ya que era lo más rápido a la mano. La otra opción era usando la gema [Carrierwave](https://github.com/carrierwaveuploader/carrierwave) y [Carrierwave Backgrounder](https://github.com/lardawge/carrierwave_backgrounder).

