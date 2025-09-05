# Lecciones Sprint 26 - F&F - Staging y Production

## Descripción de los Entornos Staging y Production con respecto a configuración y servicios

Desde el día 7 de Mayo, decidí optar por la forma de siempre de tener los entornos *staging* y *production* claramente definidos en variables `RAILS_ENV` y archivos.

|                       | **Staging**                                                                | **Production**                                                         |
| --------------------- | -------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **Subdominio**        |                                                                            | [https://adm.feedandfit.co](https://adm.feedandfit.co)                 |
| **Subdominio Heroku** | [https://feedandfitst.herokuapp.com/](https://feedandfitst.herokuapp.com/) | [https://feedandfit.herokuapp.com/](https://feedandfit.herokuapp.com/) |
| **RAILS_ENV**         | staging                                                                    | production                                                             |
| **Cloudinary**        | Con Heroku                                                                 | Mismo que staging, no importa mucho                                    |
| **Sentry**            | Cuenta Dev As Pros                                                         | Mismo que staging                                                      |
| **Mailgun**           | Con Heroku                                                                 | Mismo que staging, no importa mucho                                    |
| **Skylight**          | Cuenta correo f.q.iw, app propia                                           | Cuenta correo f.q.iw, app propia                                       |


**Heroku Apps, cambiar nombre**
Al [cambiar el nombre de una aplicación en Heroku](https://devcenter.heroku.com/articles/renaming-apps):


- Su URL también cambia
- La URL del repositorio también cambia
- Si había un dominio personalizado asociado, hay que actualizar el registro CNAME configurado
## Crear backup de Heroku Postgres de forma no manual

La idea detrás de estos comandos es hacer backups de una base de datos en Heroku Postgres sin necesidad de pagar por una instancia más cara.

Para crear un backup manual se puede usar el siguiente comando:


    $ heroku pg:backups:capture --remote staging


> Para Feed & Fit toca especificar siempre el remoto o la app con `--remote` y `--app`

Para listar los backups se ejecuta el comando:


    $ heroku pg:backups --remote staging

Algo muy interesante es que se pueden [agendar backups](https://devcenter.heroku.com/articles/heroku-postgres-backups#scheduling-backups) para evitar tener que hacer esto manualmente y no hay que pagar un peso más.


    $ heroku pg:backups:schedule DATABASE_URL --at '01:00 America/Bogota' --remote staging
## Heroku [pg:pull](https://devcenter.heroku.com/articles/heroku-postgresql#pg-push-and-pg-pull) usa el nombre de la instancia de Heroku Postgres y no la URL de la base de datos

Cuando intenté ejecutar el comando `heroku pg:pull` de la siguiente forma:


    hku pg:pull \
      postgres://fsisazzszskrro:5233ebf1a00ccdc3f946f2fa2c495599f8d4ac433dc13fd09bba51d30cfa0e50@ec2-23-23-242-163.compute-1.amazonaws.com:5432/df5fm4gd7lai8k \
      postgres://gvmtvlearkxqmu:5547219f875146bd5967914987d4a165377779a49be1c24739bb15d443ad4f2c@ec2-23-23-173-30.compute-1.amazonaws.com:5432/d18e2tsnjoju6p \
      --remote staging

Arrojaba el error:


    ▸    Unknown database: postgres://fsisazzszskrro:5233ebf1a00ccdc3f946f2fa2c495599f8d4ac433dc13fd09bba51d30cfa0e50@ec2-23-23-242-163.compute-1.amazonaws.com:5432/df5fm4gd7lai8k.
     ▸    Valid options are: DATABASE_URL

Para que ejecutara con éxito tuve que correrlo así:


    hku pg:pull \
      postgresql-angular-52619 \
      postgres://gvmtvlearkxqmu:5547219f875146bd5967914987d4a165377779a49be1c24739bb15d443ad4f2c@ec2-23-23-173-30.compute-1.amazonaws.com:5432/d18e2tsnjoju6p \
      --remote staging


## Mejorar el plan de una base de datos postgresql en Heroku

El proceso resulta que no es solo presionar un botón y listo. Tiene dos formas:


- [Con](https://devcenter.heroku.com/articles/updating-heroku-postgres-databases) [*follower*](https://devcenter.heroku.com/articles/updating-heroku-postgres-databases) pero solo sirve para los planes Standard, Premium, Private, or Shield
- [Con](https://devcenter.heroku.com/articles/upgrading-heroku-postgres-databases#upgrading-with-pg-copy) `[pg:copy](https://devcenter.heroku.com/articles/upgrading-heroku-postgres-databases#upgrading-with-pg-copy)` que sirve para todos los planes y en específico, los *Hobby-tier*

