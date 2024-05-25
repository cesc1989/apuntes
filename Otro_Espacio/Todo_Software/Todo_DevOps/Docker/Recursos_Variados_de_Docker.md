# Recursos Variados de Docker

## General
- [Tutorial Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-getting-started)
- Qué es Docker: [Docker](http://docs.docker.com/engine/introduction/understanding-docker/)
- Instalación Ubuntu: [Docker docs](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/)
- Ubuntu friendly for docker: [github](https://github.com/jeckhart/baseimage-docker) and [phusion original](https://hub.docker.com/r/phusion/baseimage/)
- [Mejores prácticas al escribir un Dockerfile](http://docs.docker.com/engine/articles/dockerfile_best-practices/)
- Aprender Docker: [Curso](http://training.docker.com/introduction-to-docker) - [Labs](http://training.play-with-docker.com/dev-landing/)
- [Difference between](https://stackoverflow.com/questions/35832095/difference-between-links-and-depends-on-in-docker-compose-yml) `[links](https://stackoverflow.com/questions/35832095/difference-between-links-and-depends-on-in-docker-compose-yml)` [and](https://stackoverflow.com/questions/35832095/difference-between-links-and-depends-on-in-docker-compose-yml) `[depends_on](https://stackoverflow.com/questions/35832095/difference-between-links-and-depends-on-in-docker-compose-yml)`


## Docker Compose
- Qué es docker compose: [Official Docs](http://docs.docker.com/compose/)
- Instalar docker compose: [Docker docs](http://docs.docker.com/compose/install/)
- [Docker Compose GitHub](https://github.com/docker/compose)
- [How to install docker-compose on Linux](https://docs.docker.com/compose/install/#install-compose)


## docker-compose commands for dockerizing rails apps

When dockerizing a rails app first build the services defined in the `docker-compose.yml` file and then recreate database:

    $ docker-compose build

After it finishes, run `rake` commands

    $ docker-compose run [SERVICE-NAME] rake db:create
    $ docker-compose run api rake db:create

Then you can start all the services as normal:

    $ docker-compose up


## How to import exported data to database in docker-compose file

When using a normal container you can import dumped data by `COPY`-ing it to the `/docker-entrypoint-initdb.d` folder in the container:

    COPY ./dump.sql /docker-entrypoint-initdb.d

When the container runs, it is going to import it back to a database.

When using docker-compose you need to define the dumped file as volume to the container service:

    services:
      postgres:
        build: ./.docker/database/
        environment:
          POSTGRES_USER: "bucket"
          POSTGRES_PASSWORD: "123456789"
        ports:
          - "5433:5432"
        volumes:
          - ./.docker/database/dump.sql:/docker-entrypoint-initdb.d/dump.sql

With help of [Stack Overflow](https://stackoverflow.com/a/43880563/1407371)


## Errores y Soluciones
- Error `Couldn't connect to Docker daemon at http+unix://var/run/docker.sock - is it running?`: [GitHub](https://github.com/docker/compose/issues/1214)
- `[invalid byte sequence in US-ASCII (Argument Error)](https://stackoverflow.com/questions/17031651/invalid-byte-sequence-in-us-ascii-argument-error-when-i-run-rake-dbseed-in-ra#17031697)` [al arrancar el comando foreman start](https://stackoverflow.com/questions/17031651/invalid-byte-sequence-in-us-ascii-argument-error-when-i-run-rake-dbseed-in-ra#17031697)


## `Local gulp not found in /app`
    $ Local gulp not found in /app
    $ Try running: npm install gulp

Answer: [Stack Overflow](https://stackoverflow.com/a/33301782/1407371)

    You need to run npm install gulp AFTER WORKDIR /app, so that gulp is installed locally in node_modules/gulp. But you already did that and having the same error. It's because in your docker-compose-dev.yml, you are mounting host directory as /app volume inside docker container. So local changes in /app directory is lost when you are running the container.
    
    You can either remove volumes from docker-compose-dev.yml or run npm install gulp in host machine.


## Docker MySQL 8 and PHP7 `cashing_sha2_password`

Cuando se intenta conectar a MySQL (versión 8) desde PHP (versión 7).
Solución:


- sha2 usa encriptación
- es incompatible
- Se puede hacer downgrade de MySQL pero a veces no sirve (?)
- Usar `mysql_native_password`: lo que hace es poner privilegios de la versión anterior de MySQL.


