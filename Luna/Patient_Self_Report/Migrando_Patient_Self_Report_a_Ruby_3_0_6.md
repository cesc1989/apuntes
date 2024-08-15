# Migrando Patient Self Report a Ruby 3.0.6

# Enlaces y Recursos

- [[Migrando_a_Ruby_3_0_2_con_WKHTMLTOPDF]]
- Apuntes sobre Docker con Alpine [+Apuntes de Docker - Parte 1](https://paper.dropbox.com/doc/Apuntes-de-Docker-Parte-1-PxjQ3IeKE8xn3T4iKDpCb) 
- Ver [versiones de Alpine y revisar paquetes](https://pkgs.alpinelinux.org/packages) disponibles
- [Imagen](https://hub.docker.com/layers/library/ruby/3.0.6-alpine3.16/images/sha256-af7cefcafaab95aa3f72c579fb96941d364145c4dd620f228f59e9f628bced68?context=explore) de Docker a usar: ruby:3.0.6-alpine3.16

# Pasos

- Instalar Ruby 3.0.6 en local
    - Correr `bundle install` ✅ 
    - Correr `rspec` ✅ 
- Crear imagen docker en local con versión de Alpine
    - Debe construir la imagen ✅ 
- Enviar a Alpha
    - Debe construir la imagen
    - Debe correr los tests


## Comando para crear la imagen en local

    docker image build \
      --build-arg SIDEKIQ_LICENSE_KEY=KEY \
      -t prueba-ruby306 \
      -f Dockerfile .


# Errores

## Error al intentar instalar sassc

    An error occurred while installing sassc (2.2.1), and Bundler cannot continue.
    
    In Gemfile:
      sass-rails was resolved to 6.0.0, which depends on
        sassc-rails was resolved to 2.1.2, which depends on
          sassc

Intenté:

    gem install sassc -v 2.2.1

pero falló:

    Building native extensions. This could take a while...
    ERROR:  Error installing sassc:
    ERROR: Failed to build gem native extension.

Lo que funcionó fue la bandera  `**-- --disable-march-tune-native**`

    gem install sassc -v 2.2.1 -- --disable-march-tune-native
    Building native extensions with: '--disable-march-tune-native'
    This could take a while...
    Successfully installed sassc-2.2.1
    Parsing documentation for sassc-2.2.1
    Installing ri documentation for sassc-2.2.1
    Done installing documentation for sassc after 1 seconds
    1 gem installed

Visto en [este issue](https://github.com/sass/sassc-ruby/issues/146#issuecomment-541364174).

