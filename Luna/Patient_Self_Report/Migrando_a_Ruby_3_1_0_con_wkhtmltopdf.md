# Migrando a Ruby 3.1.0 con wkhtmltopdf

# Error de zeitwerk y net/smtp

Instalé Ruby 3.1.0, corrí bundle y luego cuando ejecuto el comando de rspec todas las pruebas se revientan con este error:

    Failure/Error: example.run
         
         LoadError:
           cannot load such file -- net/smtp
         # /Users/francisco/.gem/ruby/3.1.0/gems/zeitwerk-2.6.0/lib/zeitwerk/kernel.rb:35:in `require'
         # /Users/francisco/.gem/ruby/3.1.0/gems/zeitwerk-2.6.0/lib/zeitwerk/kernel.rb:35:in `require'
         # /Users/francisco/.gem/ruby/3.1.0/gems/mail-2.7.1/lib/mail.rb:9:in `<module:Mail>'
         # /Users/francisco/.gem/ruby/3.1.0/gems/mail-2.7.1/lib/mail.rb:3:in `<top (required)>'

La solución es instalar la gema ‘net-smtp’:

    gem 'net-smtp', require: false

Este mismo error puede pasar pero con la variante:

    zeitwerk-2.6.0/lib/zeitwerk/kernel.rb:35:in `require': cannot load such file -- net/pop (LoadError)
    
    zeitwerk-2.6.0/lib/zeitwerk/kernel.rb:35:in `require': cannot load such file -- net/imap (LoadError)

Parece que tiene que ver con la gema mail.

Visto en [Stack Overflow](https://stackoverflow.com/questions/70500220/rails-7-ruby-3-1-loaderror-cannot-load-such-file-net-smtp).


# Error usando wkhtmltopdf

Salió este error ya tratando de generar un PDF desde código:

    Failed to execute:
    ["/usr/bin/wkhtmltopdf", "--viewport-size", "1024x768", "--lowquality", "file:////tmp/wicked_pdf20230608-1-dppslw.html", "/tmp/wicked_pdf_generated_file20230608-1-18qoy1.pdf"]
    Error: PDF could not be generated!
     Command Error: The switch --viewport-size, is not support using unpatched qt, and will be ignored.QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-puma'
    Loading page (1/2)

Las sugerencias en internet eran reinstalar wkhtmltopdf desde las fuentes pero como esto es Alpine la cosa no me cuadraba.

La solución la obtuve al instalar varias de las dependencias actuales que tiene el proyecto de Therapist Signup

     libxrender libxext gcompat fontconfig freetype ttf-dejavu ttf-droid ttf-freefont ttf-liberation

Alguna de esas hizo el migralo 🤣 

