# CI, CD: Jenkins, Rails & Bitbucket

## CI y CD
- CodeDeploy [deep dive](https://docs.google.com/presentation/d/19r3BCzBmFViJP3YHisn4miJBz2Oq0EH4bbpVDhjWZXQ/edit#slide=id.gb5d780cf7_0_1)


## Para Aplicaciones Rails
- Al cargar la base de datos para tests, normalmente hacía `rake db:setup` o correr migraciones y etc, pero hay que tener en cuenta que en un entorno CI, **solo importa el ambiente test**, por eso solo debería correrse `rake db:test:prepare`: [Sobre el comando en Stack Overflow](https://stackoverflow.com/questions/15169894/what-does-rake-dbtestprepare-actually-do#15170001)
- **RECUERDA** que en el archivo `config/database.yml` hay que cuadrar la variable de entorno `DATABASE_URL` que es la que da el servicio postgres.


## Pipelines de Bitbucket
- Configurar proyecto Rails/Ruby con pipelines: [Docs](https://confluence.atlassian.com/bitbucket/ruby-with-bitbucket-pipelines-872005618.html)
- [Validador de archivo pipelines](https://confluence.atlassian.com/bitbucket/ruby-with-bitbucket-pipelines-872005618.html)
- Como instalar dependencias en el contenedor que corre las tareas: [Docs](https://confluence.atlassian.com/bitbucket/specify-dependencies-in-your-pipelines-build-873928908.html)
- Usar postgres como servicio para el contenedor que corre los tests: [Comentario](https://community.atlassian.com/t5/Bitbucket-questions/RSpec-pipeline-configurations-for-Ruby-on-Rails/qaq-p/155412#M22587) - [Docs](https://confluence.atlassian.com/bitbucket/how-to-run-common-databases-in-bitbucket-pipelines-891130454.html#HowtoruncommondatabasesinBitbucketPipelines-PostgreSQL%E2%80%93testuser)
- Todo sobre el formato de bitbucket-pipelines.yml: [Docs](https://confluence.atlassian.com/bitbucket/configure-bitbucket-pipelines-yml-792298910.html)
- Cómo usar servicios y bases de datos: [Docs](https://confluence.atlassian.com/bitbucket/use-services-and-databases-in-bitbucket-pipelines-874786688.html)
- En el repo del proyecto Feed & Fit hay un pipeline bien configurado para correr tests

