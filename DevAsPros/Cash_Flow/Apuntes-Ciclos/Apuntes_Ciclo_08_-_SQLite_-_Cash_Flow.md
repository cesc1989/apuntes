# Apuntes Ciclo 08 - SQLite - Cash Flow

# Ecosistema de SQLite es interesante

En este [tuit](https://twitter.com/lazaronixon/status/1684927993158742016) de Lazaro (trabaja en Koombea/Luna) que menciona ventajas de usar MySQL sobre PostgreSQL, una persona menciona que SQLite también tiene buenas cosas.

Menciona y encontré:

- Litestack
- Litestream
- LiteFS
## Litestream

[Sitio web](https://litestream.io/).

> Fully-replicated database with no pain and little cost.

**OJO**: aquí sugieren como alternativa hacer backup de la base de datos PERO NO USANDO el comando `cp`. Y esa es la forma en que lo tengo hecho en Cashflow 🥲 

[Guía para hacer cron de base de datos SQLite](https://litestream.io/alternatives/cron/).


## LiteFS

[Sitio web](https://github.com/superfly/litefs).

> LiteFS is a FUSE-based file system for replicating SQLite databases across a cluster of machines.


## Litestack

[Sitio web](https://github.com/oldmoe/litestack).

> litestack is a revolutionary gem for Ruby and Ruby on Rails that provides an all-in-one solution for web application development. It exploits the power and embeddedness of SQLite to include a full-fledged SQL database, a fast cache and a robust job queue all in a single package.

Básicamente con esta gema se reemplazan las gemas:

- sqlite3
- sidekiq
- redis
- action_cable

Todo usando SQLite como base de datos.

# Librería UI Tabler. Reemplazo a AdminLTE

Encontré [Tabler](https://tabler.io/). Una plantilla basada en Bootstrap que es gratis! Está muy bacana y podría tomarla para guiarme y mejorar la UI de Cashflow.

# Usar helpers de vistas en los tests

Se incluyen en el archivo de pruebas:

    include ActionView::Helpers
    include ActionView::Helpers::NumberHelper

Visto en [Stack Overflow](https://stackoverflow.com/a/14402860/1407371).

# ¿Es válido llamar a un modelo directo en la vista?

Me pareció importante esta pregunta y respuesta sobre “cargar datos del modelo en la vista” [https://stackoverflow.com/questions/25984718/is-it-ever-ok-to-call-a-model-class-method-from-within-a-view](https://stackoverflow.com/questions/25984718/is-it-ever-ok-to-call-a-model-class-method-from-within-a-view)

Conclusiones:

- No tiene que ser un problema
- Se pueden usar clases para aislar y no sobrecargar la vista
# Dar formato a enteros como moneda con JavaScript

La forma para mostrar un entero como moneda con JavaScript.

    amount.toLocaleString('en-US', { style: 'currency', currency: 'USD' });

Y para quitar fracciones se usa la opción

    minimumFractionDigits: 0

Visto en [Stack Overflow](https://stackoverflow.com/a/26745078/1407371).


# Hacer filtro con .joins()

Para poder usar el campo de una tabla asociada hay que usar el método `.joins` para luego accederlo en la clausula where:

    def search
        Expenditure.includes(:category)
                   .joins(:category)
                   .where(user_id: current_user.id)
                   .where("LOWER(description) LIKE '%#{params[:query]}%' OR LOWER(categories.name) LIKE '%#{params[:query]}%'")
      end


# Ejecutar workflows en Github actions reusando jobs

Hay que usar una combinación de varias instrucciones:

- jobs - uses → [https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_iduses](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_iduses)
- jobs - needs → [https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idneeds](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idneeds)
- y on.workflow_call → [https://docs.github.com/en/actions/using-workflows/reusing-workflows#creating-a-reusable-workflow](https://docs.github.com/en/actions/using-workflows/reusing-workflows#creating-a-reusable-workflow)
- jobs.secrets.inherit [https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idsecretsinherit](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idsecretsinherit)

Así quedó el workflow:

    name: Run tests and deploy via SSH
    on: [push, pull_request]
    
    jobs:
      tests:
        name: RSpec
        runs-on: ubuntu-latest
        services:
          postgres:
            image: postgres:latest
            ports:
              - 5432:5432
            options: (...)
            env: (...)
        steps:
          - uses: actions/checkout@v3
          - uses: ruby/setup-ruby@v1
            with:
              ruby-version: 3.1.0
              bundler-cache: true
          - name: Install postgres client
            run: sudo apt-get install libpq-dev
          - name: Prepare database
            run: bundler exec rails db:prepare RAILS_ENV=test SECRET_KEY_BASE=secret_key_base_test
          - name: Run tests
            env:
              SECRET_KEY_BASE: secret_key_base_test
            run: bundler exec rspec
    
      deploy:
        name: Deploy
        needs: [tests]
        uses: ./.github/workflows/deploy.yml
        secrets: inherit

