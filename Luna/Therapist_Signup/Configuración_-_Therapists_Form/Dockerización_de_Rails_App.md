# Dockerización de Rails App

Relacionado

- [Ruby on Whales: Dockerizing Ruby and Rails development](https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development)
    - [Repo con los archivos](https://github.com/evilmartians/terraforming-rails)

# ¿Qué es `DEBIAN_FRONTEND=noninteractive`?

`DEBIAN_FRONTEND=noninteractive` está explicado en [esta respuesta](https://askubuntu.com/questions/972516/debian-frontend-environment-variable/972528#972528).


    noninteractive
              This is the anti-frontend. It never interacts with you  at  all,
              and  makes  the  default  answers  be used for all questions. It
              might mail error messages to root, but that's it;  otherwise  it
              is  completely  silent  and  unobtrusive, a perfect frontend for
              automatic installs. If you are using this front-end, and require
              non-default  answers  to questions, you will need to preseed the
              debconf database; see the section below  on  Unattended  Package
              Installation for more details.

# ¿Qué significa la opción `-qq` en `apt-get update`?

`apt-get update --qq` significa doblemente *quiet*. Ver la [*man page*](https://linux.die.net/man/8/apt-get) [al respecto](https://linux.die.net/man/8/apt-get).

# Construcción *multi stage* de imagen

Resulta que el despliegue de la aplicación *Therapists Credentialing Frontend* que está configurado con Docker tiene un *Dockerfile* que luce así:

    FROM tiangolo/node-frontend:10 as build-stage
    WORKDIR /app
    COPY package*.json /app/
    RUN npm install
    COPY ./ /app/
    RUN npm run build
    FROM nginx:1.15
    COPY --from=build-stage /app/build/ /usr/share/nginx/html
    COPY --from=build-stage /nginx.conf /etc/nginx/conf.d/default.conf

Cuando buscaba el contenedor y entraba en este, no veía el contenido del aplicación. Es más, no encontraba la carpeta `/app/`. Luego de buscar y probar, me di cuenta que la carpeta donde está alojada la aplicación es `/usr/share/nginx/html`.

Tal cambio ocurre en la línea 8:

    COPY --from=build-stage /app/build/ /usr/share/nginx/html

En la documentación de la instrucción `COPY` no explican mucho al respecto de la opción `--from` sin embargo, en la sección [**Use multi-stage builds**](https://docs.docker.com/develop/develop-images/multistage-build/) ****sí explican al respecto.

La construcción de imágenes por fases permite generar imágenes más livianas usando el resultado de la anterior.

En este caso, la imagen que empieza con la línea:

    FROM tiangolo/node-frontend:10 AS build-stage

La instrucción `AS` le da nombre al `FROM tiangolo/node-frontend:10`. Todo lo que ocurre de ahí hasta el siguiente `FROM` queda en la capa pero en la siguiente fase de la construcción solo se usa el producido:

    FROM nginx:1.15
    COPY --from=build-stage /app/build/ /usr/share/nginx/html
    COPY --from=build-stage /nginx.conf /etc/nginx/conf.d/default.conf

Por lo que la imagen final será mucho más liviana y tendrá lo necesario.

# Pasando Argumentos al Dockerfile

Buscando pasar `SECRETE_KEY_BASE` al proceso de despliegue del formulario de *Credentialing* para poder ejecutar el comando `rake assets:precompile`. Como todo esto es con Docker la cuestión era algo nueva para mí y consiste de varios pasos.

Un paso era agregar la variable en la [lista de variables de entorno](https://circleci.com/docs/2.0/configuration-reference/#environment) del archivo `config.yml` de CircleCI.

    deploy:
        working_directory: ~/therapist-credentialing-backend
        docker:
          - image: koombea/circleci:0.11.11
        environment:
          ENVIRONMENT: staging
          CIRCLE_PROJECT_REPONAME: luna-api
          SECRET_KEY_BASE: d71413b6d10aede9e10f9cfe363adb38cf3e4d4da4de6cc17578b8f52bf013f06046905a241e4e238902ae429bf4a9bc474d9807e9af577af2e6e8bc230fb1fb
        steps:
          - checkout
          - setup_remote_docker
          - run:

Dicha variable se puede usar en el shell script en la instrucción `run`.

    - run:
      name: Building Docker image for release
      command: |
       # (...)
        docker build --cache-from=${DOCKER_REGISTRY}/${CIRCLE_PROJECT_REPONAME}-${ENVIRONMENT}:${CACHE} \
        --build-arg SECRET_KEY_BASE=${SECRET_KEY_BASE} \
        -t ${DOCKER_REGISTRY}/${CIRCLE_PROJECT_REPONAME}-${ENVIRONMENT}:${CIRCLE_BRANCH//\//_}_${CIRCLE_BUILD_NUM} .
       # (...)

Según la [documentación de Docker](https://docs.docker.com/engine/reference/builder/#arg), el Dockerfile debe definir las variables de entorno disponibles durante la creación de la imagen.

De este modo, en el archivo `Dockerfile` definí la variable `SECRET_KEY_BASE`.

    # (...)
    
    ARG SECRET_KEY_BASE
    
    ENV BUNDLE_WITHOUT="development test" \
        APP=/usr/src/app/ \
        RAILS_ENV=production \
        BUILD_PACKAGES="build-base" \
        DEV_PACKAGES="git libxml2-dev libxslt-dev" \
        RUNTIME_PACKAGES="postgresql-dev tzdata nodejs"
    
    # (...)
    
    RUN rails assets:precompile
    
    ENTRYPOINT ["bin/entrypoint"]
    
    CMD ["rails", "server"]
    

Y así, finalmente poder ejecutar el comando que me interesaba `rails assets:precompile`.

# Build para proyecto Rails 6 con Webpacker

En resumen, este es el Dockerfile deseado cuando se usan imágenes de Alpine.

    FROM ruby:2.5.3-alpine
    
    LABEL Author="Andrews Herrera <andrews.herrera@koombea.com>"
    
    ENV RAILS_LOG_TO_STDOUT=enabled \
        RAILS_SERVE_STATIC_FILES=enabled \
        BUNDLE_WITHOUT="development test" \
        APP=/usr/src/app/ \
        RAILS_ENV=production \
        PACKAGES="build-base postgresql-dev tzdata nodejs git libxml2-dev libxslt-dev openssl yarn"
    
    WORKDIR $APP
    COPY Gemfile* $APP
    COPY package*.json yarn.lock $APP
    
    RUN set -ex && \
        apk add --no-cache --virtual .packages $PACKAGES && \
        gem install bundler && \
        bundle config set without $BUNDLE_WITHOUT && \
        yarn install && \
        bundle install --jobs 2 --retry 5
    
    RUN apk -Uuv add groff less python py-pip
    RUN pip install awscli
    RUN apk --purge -v del py-pip
    RUN rm /var/cache/apk/*
    
    COPY . $APP
    
    RUN bundle exec rails assets:precompile
    
    ENTRYPOINT ["./bin/entrypoint"]
    
    CMD ["bundle", "exec", "rails", "s", "-b", "0.0.0.0"]
    

Nota que se instala `yarn` con la lista de `PACKAGES`, se copia el `package.json` y también el `yarn.lock`, y se ejecuta el comando `yarn install` con normalidad.

No se olvide correr `bundle exec rails assets:precompile`. Me dio confianza estos pasos cuando vi estos proyectos de Dockerización de Rails 6 con Webpacker:

- [Docker Rails](https://github.com/ledermann/docker-rails/blob/develop/Dockerfile)
    - Menciona que usa la imagen de abajo en el Dockerfile
- [Docker Rails Base](https://github.com/ledermann/docker-rails-base/blob/master/Builder/Dockerfile)



