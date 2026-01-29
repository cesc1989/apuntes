# Apuntes Ciclo 03 - Despliegue - Cash Flow
La mayor enseñanza de este ciclo es no usar Puma si me toca configurarlo a mí en un VPS.

Es muy molesto de configurar. Passenger es muchísimo más sencillo y queda demostrado por todas las veces que lo he hecho ya:

- Macsa
- Bucket
- Brilla
- Cash Flow
# ¿Cómo cargar la llave privada mediante Packer?

No se pudo.

En todo caso, comando para subir la llave privada manualmente:

    scp -i /Users/francisco/.ssh/linode /Users/francisco/.ssh/gh_ubuntu ubuntu@139.144.196.192:~/.ssh/gh_ubuntu


# Haciendo despliegue manual

Me tocó hacer el archivo de configuración ssh para poder tener acceso Github. Lo hice según [esta guía](https://gist.github.com/cesc1989/ce791228177867271147770629fe754b).

    $ nano ~/.ssh/config
    
    $ cat ~/.ssh/config
    Host gh
    HostName github.com
    User git
    IdentityFile ~/.ssh/gh_ubuntu


## Estructura de carpetas

Esta es la estructura final de carpetas para ejecutar la aplicación.

    cashflow/
    ├── app
    │ └── vendor
    ├── backups
    ├── db
    │ └── cashflow_production.sqlite # archivo master importante
    └── deployments
        ├── api-gems
        └── api-release

**En app tocó crear vendor porque al inicio está vacía.**


## NVM no se instala al provisionar con Packer

Tocó instalar manualmente

    nvm install --lts

**Actualiza el symlink de node para que tenga la misma versión**

    cat ~/.clave | sudo -S ln -sfv /home/ubuntu/.nvm/versions/node/v20.12.2/bin/node /usr/local/bin/node


## Revisión de conf de nginx es con sudo
    sudo nginx -t
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful


## ENVs y ~.profile

Toca hacer export de las variables en el .profile o pasar el argumento en cada comando:

    RAILS_ENV=$RAILS_ENV SECRET_KEY_BASE=$SECRET_KEY_BASE rake

o configurar bien el .profile

    export ENV=value


## Passenger > Puma

El log de error de passenger está aquí: `/var/log/nginx/error.log`

Sobre los logs de passenger [https://www.phusionpassenger.com/docs/advanced_guides/troubleshooting/nginx/log_file.html](https://www.phusionpassenger.com/docs/advanced_guides/troubleshooting/nginx/log_file.html)

**Comandos Passenger**

    # passenger necesita node a nivel global
    sudo ln -sf /home/ubuntu/.nvm/versions/node/v18.16.1/bin/node /usr/local/bin/node
    
    # reinicia passenger y nginx
    sudo service nginx restart
    
    # revisa la instalación
    sudo /usr/bin/passenger-config validate-install

Más info al respecto de Passenger y NodeJS [https://www.phusionpassenger.com/library/config/nginx/reference/#passenger_nodejs](https://www.phusionpassenger.com/library/config/nginx/reference/#passenger_nodejs)

