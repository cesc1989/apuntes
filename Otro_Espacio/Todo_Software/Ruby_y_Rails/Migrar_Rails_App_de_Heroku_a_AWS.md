# Migrar Rails App de Heroku a AWS

## En Heroku
- Revisar servicios en uso
- Obtener copia de la aplicación

**Descargar backup de la base de datos**

- Se puede hacer desde la interfaz de Heroku. Yendo a Heroku Postgres, sección “Durability” y haciendo el backup manual.
- Si se tiene el [CLI de Heroku](https://devcenter.heroku.com/articles/heroku-postgres-backups#creating-a-backup) con `heroku pg:backups:capture --app APP_NAME`
- Y se descarga con `heroku pg:backups:download`


## En el Proyecto

**Copiar Scripts de Despliegue**
Del repositorio *Backend Stuff*, copiar los archivos en una carpeta llamada `scripts`(por conveniencia).

    $ mkdir scripts
    
    $ wget -O ./scripts/deploy_api.sh https://raw.githubusercontent.com/cesc1989/backendstuff/master/deployment/deploy_api.sh
    
    $ wget -O ./scripts/pull_repo.sh https://raw.githubusercontent.com/cesc1989/backendstuff/master/deployment/pull_repo.sh
    
    $ wget -O ./scripts/after_deploy.sh https://raw.githubusercontent.com/cesc1989/backendstuff/master/deployment/after_deploy.sh
    
    $ wget -O ./scripts/application_start.sh https://raw.githubusercontent.com/cesc1989/backendstuff/master/deployment/application_start.sh


## En AWS EC2
- Crear instancia en consola de AWS
    - Idealmente, con Ubuntu 18.04
- Guardar llave SSH


## En el servidor EC2

**Instalar git**

    $ sudo apt-get install git

**Clonar repositorio backendstuff**

    $ git clone https://github.com/cesc1989/backendstuff.git

**Instalar dependencias necesarias**

    $ sudo bash ~/backendstuff/dependencies/install_lib_essentials.sh

**Instalar RVM y Ruby**

    $ sudo bash ~/backendstuff/dependencies/install_rvm.sh

**Instalar NVM y NodeJS**

    $ sudo bash ~/backendstuff/dependencies/install_nvm.sh

**Instalar Nginx**
Idealmente, con Phussion Passenger por mejor integración con Nginx y porque jode menos que Puma al configurar el virtual host.

    $ sudo bash ~/backendstuff/dependencies/install_passenger_nginx.sh

**Crear Estructura de Carpetas**
Esta estructura es sencilla y sirve para el despliegue y ejecutar la aplicación.

    # ~/app
    # ├── api
    # └── deployments
    #   ├── api-gems
    #   └── api-release
    
    $ mkdir -p app/api
    $ mkdir -p app/deployments/api-gems
    $ mkdir -p app/deployments/api-release

**Crear virtual host de Nginx**
Alojar en la carpeta `config/` del proyecto y commitear.

    server {
      listen 80;
      listen [::]:80;
      server_name [IP_DEL_SERVIDOR];
    
      root /home/ubuntu/app/api/public;
      access_log off;
      error_log /home/ubuntu/app/api/log/nginx.error.log warn;
    
      passenger_enabled on;
      passenger_app_env production;
      passenger_ruby /home/ubuntu/.rvm/gems/ruby-2.5.3/wrappers/ruby;
    
      error_page 500 502 503 504 /500.html;
      client_max_body_size 10M;
      keepalive_timeout 10;
    }

**Configurar llave SSH para hacer pull al repositorio**
Crea una nueva llave SSH y guardarla en tu cuenta de GitHub o Bitbucket y súbela al servidor de EC2.

    $ scp ~/.ssh/[LLAVE_A_SUBIR] ubuntu@[IP]:~/.ssh/

Luego, en el servidor EC2, configura dicha llave para comunicarse con GitHub o Bitbucket.

    $ touch ~/.ssh/config
    $ nano ~/.ssh/config
    
    Host github.com
      Hostname github.com
      IdentityFile /home/ubuntu/.ssh/[THE-KEY-YOU-GENERATED]
      User git
    
    Host bitbucket.org
      Hostname bitbucket.org
      IdentityFile /home/ubuntu/.ssh/[THE-KEY-YOU-GENERATED]
      User git

Probar la conexión con los comandos:

    $ ssh -t git@github.com
    $ ssh -t git@bitbucket.org


## En AWS RDS
- Crear instancia
- Guardar datos de base datos
- Cargar backup


## En Proyecto y Circle CI

**Configurar nueva URL de BD en producción**
En el archivo `~/.profile` ingresar las variables de entorno necesarias

    $ nano ~/.profile

**Configurar acceso a Circle CI en repo**
Iniciar sesión con GitHub o Bitbucket y dar permisos para acceder al respositorio.

**Configurar archivo de Circle CI**
Crear carpeta `.circleci` en raíz del proyecto y en ella el archivo `config.yml`

    $ mkdir ./.circleci
    $ touch ./.circleci/config.yml

Copiar esta configuración

    version: 2
    jobs:
      deploy:
        machine:
          enabled: true
        steps:
          - run:
              name: Deploy over SSH
              command: |
                ssh $SSH_USER@$SSH_HOST "/home/ubuntu/app/deploy_api.sh"
    workflows:
      version: 2
      deployment:
        jobs:
          - build
          - deploy:
              filters:
                branches:
                  only: master
    

**Configurar acceso al servidor EC2 en Circle CI**
[Ver Deployment Examples SSH](https://circleci.com/docs/2.0/deployment-examples/#ssh).

En la sección de variables de entorno hay que crear dos nuevas variables.

- `SSH_USER` → el nombre de usuario con el que se ingresa al servidor EC2.
    - *ubuntu* 
- `SSH_HOST` → la IP del servidor EC2

