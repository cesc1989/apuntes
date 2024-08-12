# Docker Build para Credentialing Frontend

Empezaré con esto:

*Virtual Machines* **>** *Docker Containers*

Resulta que a una configuración que funciona, al hacerle un cambio mínimo, el *build* empezó a fallar en Circle CI con este mensaje:

> "docker build" requires exactly 1 argument.
> See 'docker build --help'

## Las Configuraciones

**Rama master:** Build no falla

    docker build -t frontend . -f Dockerfile-prod \
    --build-arg REACT_APP_REAL_API_SERVER=$REACT_APP_REAL_API_SERVER

**Rama staging:** Build fallaba

    docker build -t frontend . -f Dockerfile-prod \
    --build-arg REACT_APP_REAL_API_SERVER=$REACT_APP_REAL_API_SERVER \
    --build-arg REACT_APP_RECAPTCHA_SECRET_KEY=$REACT_APP_RECAPTCHA_SECRET_KEY

¿Cómo lo corregí? Dejando de último el `PATH` al Dockerfile. Como [indica esta respuesta](https://stackoverflow.com/a/46517290/1407371).

    docker build --build-arg REACT_APP_REAL_API_SERVER=$REACT_APP_REAL_API_SERVER \
                  --build-arg REACT_APP_RECAPTCHA_SITE_KEY=$REACT_APP_RECAPTCHA_SITE_KEY \
                  -t frontend -f Dockerfile-prod .

En algún momento probé ponerle comillas a las variables del `--build-arg` porque una tiene un guión pero ese parece que no fue el problema. Aunque podría haberlo sido [si hubiera espacios en los valores](https://stackoverflow.com/a/56194139/1407371).

## Conclusión

Todo esto solo me hace querer más las máquinas virtuales sobre los contenedores.

## Temas similares

[[Integración_con_AWS_S3_y_Docker_-_Credentialing_Fo]]

[[Caso_Generación_de_PDF_WKHTMLTOPDF_y_Docker]]

