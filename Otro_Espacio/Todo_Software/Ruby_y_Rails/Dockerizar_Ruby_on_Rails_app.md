# Dockerizar Ruby on Rails app

## Híbrido: Docker solo para la base de datos
> Tomado del [artículo de Jason Swet](https://www.codewithjason.com/hybrid-approach-dockerizing-rails-applications/)

Cuando aprendí Docker, vi las ventajas para dockerizar entornos de desarrollo. Sin embargo, esto también trae varios inconvenientes a la hora de trabajar con una app Rails totalmente dockerizada:


- Correr los comandos de Rails es más lento. Incluso hay que ejecutar un comando más largo
    $ rails db:migrate
    
    $ docker-compose run web rails db:migrate


- La aplicación en el navegador irá más lento.
- Ejecutar los tests también es un inconveniente. Comando más extenso y ejecución lenta pero también complicado correr e2e tests descabezados.
- Complicado usar herramientas como `binding.pry` o Byebug `debugger`.

Por eso, una forma híbrida es lo mejor. Se instala Ruby y Rails de manera nativa en el computador y se deja Docker para la base de datos.

Cosas nativas:

- RVM
- Ruby
- Rails
- Webpack

Cosas dockerizadas:

- PostgreSQL
- Redis


    ---
    # Docker Compose 2.4 is for local development
    # https://www.heroku.com/podcasts/codeish/57-discussing-docker-containers-and-kubernetes-with-a-docker-captain - Source on that.
    version: '2.4'
    
    services:
      postgres:
        image: postgres:13.1-alpine
        mem_limit: 256m
        volumes:
          - postgresql:/var/lib/postgresql/data:delegated
        ports:
          - "127.0.0.1:5432:5432"
        environment:
          PSQL_HISTFILE: /root/log/.psql_history
          POSTGRES_USER: mednote_development
          POSTGRES_PASSWORD: pgpassword
        restart: on-failure
        healthcheck:
          test: ["CMD-SHELL", "pg_isready -U postgres"]
          interval: 10s
          timeout: 2s
          retries: 10
        logging:
          driver: none
    
      redis:
        image: redis:4.0.14-alpine
        mem_limit: 64m
        volumes:
          - redis:/data:delegated
        ports:
          - "127.0.0.1:6379:6379"
        restart: on-failure
        logging:
          driver: none
    
    volumes:
      postgresql:
      redis:
      storage:

Y el `Procfile.dev` de la aplicación sería como:

    docker:    docker-compose up
    web:       bundle exec puma -p 3000
    worker:    bundle exec sidekiq
    webpacker: bundle exec bin/webpack-dev-server

