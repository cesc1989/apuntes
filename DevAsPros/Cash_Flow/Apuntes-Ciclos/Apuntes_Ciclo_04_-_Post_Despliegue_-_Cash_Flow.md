# Apuntes Ciclo 04 - Post Despliegue - Cash Flow

# Configuración correcta de archivo .sqlite

## Sospecha #1

Algo está dañando el archivo al hacer rsync.

Resulta que al ejecutar migraciones se está usando el archivo que está en
```
cashflow/deployments/api-release/db/cashflow_production.sqlite
```

Y luego la app se ejecuta en otra carpeta que también tiene un archivo de base de datos
```
cashflow/app/db/cashflow_production.sqlite
```

**Abrir bd en sqlite3**
```bash
sqlite3 cashflow/deployments/api-release/db/cashflow_production.sqlite
```

## ¿Qué hacer?

- Procurar que ese archivo se sincronice
- Cambiar la instrucción de `rsync a --delete-after`

## Revisión

En `app/db/cashflow_production.sqlite`:
```sql
sqlite> select count(1) from categories;
    81
```

En `cashflow/deployments/api-release/db/cashflow_production.sqlite`
```sql
sqlite> select count(1) from categories;
    0
```

## Después de despliegue

Lo que va a pasar es que se está sincronizando el archivo de la carpeta de despliegue con la carpeta de app. Entonces se pierden todos los datos.

Veamos:

En `app/db/cashflow_production.sqlite`:
```sql
sqlite> select count(1) from categories;
    0
```

En En `cashflow/deployments/api-release/db/cashflow_production.sqlite`
```sql
sqlite> select count(1) from categories;
    0
```

## ¿Solución?

Poner el archivo en una carpeta por fuera de todo el flujo de migración y rsync. Luego apuntarla en el database.yml

## Pruebas

Moví el archivo de la carpeta db a `~/cashflow/db` y quité el de la carpeta releases.

Luego, en el database.yml apunté a el archivo que será único.

1. Primera prueba funcionó: luego de un despliegue las migraciones ejecutaron y las rake task también.
2. Segunda prueba funcionó: navegué en el sitio y cree registros


# Instalar certificados SSL con Certbot

Etiquetas: #despliegue_vps 

*NOTA: no enlaces los certificados en las directivas del archivo nginx o habrá error de configuración en el servidor.*

Comandos instalación de Certbot:
```bash
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Se usa así:
```bash
sudo certbot
```

Ubicación de los certificados:
```bash
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/cashflow.devaspros.com/fullchain.pem
Key is saved at:   /etc/letsencrypt/live/cashflow.devaspros.com/privkey.pem
This certificate expires on 2023-10-12.
```

## Uso de Certbot para generar certificados

Tiene que estar el dominio/subdominio ya configurado con nginx

```bash
sudo certbot

[sudo] password for ubuntu: 

Saving debug log to /var/log/letsencrypt/letsencrypt.log

  

Which names would you like to activate HTTPS for?

We recommend selecting either all domains, or all domains in a VirtualHost/server block.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

1: cashflow.devaspros.com

2: coshinotes.devaspros.com

3: enlacito.co

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

Select the appropriate numbers separated by commas and/or spaces, or leave input

blank to select all options shown (Enter 'c' to cancel): 3

Requesting a certificate for enlacito.co

  

Successfully received certificate.

Certificate is saved at: /etc/letsencrypt/live/enlacito.co/fullchain.pem

Key is saved at:         /etc/letsencrypt/live/enlacito.co/privkey.pem

This certificate expires on 2025-08-08.

These files will be updated when the certificate renews.

Certbot has set up a scheduled task to automatically renew this certificate in the background.

  

Deploying certificate

Successfully deployed certificate for enlacito.co to /etc/nginx/sites-enabled/nginx.enlacito.production.conf

Congratulations! You have successfully enabled HTTPS on https://enlacito.co

  

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

If you like Certbot, please consider supporting our work by:

 * Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate

 * Donating to EFF:                    https://eff.org/donate-le

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```


# Linode CLI y backup a Object Storage

Hay que instalar pip3

Comandos:
```bash
sudo apt install -y python3-pip
pip3 --version
```

Comandos para el CLI:
```bash
pip3 install linode-cli --upgrade

pip3 install boto3
```

## Probar listar en el bucket

El comando no es linode-cli object-storate sino
```
$ linode-cli obj ls
2023-07-14 21:18  cashflow-backups
```

El access token que se configure debe tener permisos de lectura en Account y de lecto-escritura para Object Storage.

## Comandos para Backup

Para listar archivos:
```bash
linode-cli obj ls cashflow-backups
									DIR    db/
2023-07-14 21:48  98304  cashflow_production.sqlite
```

Para hacer backup:
```bash
linode-cli obj put ./cashflow/db/cashflow_production.sqlite cashflow-backups
Uploading cashflow_production.sqlite:
 |####################################################################################################| 100.0%
Done.
```


# Configurar acceso ssh en una GitHub Action 🔑

Etiquetas: #despliegue_vps

> [!Note]
> Se pueden usar **las mismas llaves** para acceder al servidor desde el computador personal.

De [este artículo](https://dev.to/andersbjorkland/how-to-deploy-with-deployer-and-github-actions-k07) tomé la forma de configurar un acceso por SSH en el runner del GitHub Action para entrar al servidor y poder luego ejecutar comandos.

Así queda la parte del acción que configura el archivo `~/.ssh/config` del contenedor del action:
```bash
steps:
  - uses: actions/checkout@v3
  - name: Configure SSH
    env:
      SSH_KEY: ${{ secrets.PRIVATE_KEY }}
      KNOWN_HOSTS: ${{ secrets.KNOWN_HOSTS }}
      SSH_HOST: ${{ secrets.TARGET_HOST }}
      SSH_USER: ${{ secrets.TARGET_USER }}
    run: |
      mkdir -p ~/.ssh/
      echo "$KNOWN_HOSTS" > ~/.ssh/known_hosts
      echo "$SSH_KEY" > ~/.ssh/deploy.key
      chmod 600 ~/.ssh/deploy.key
      cat >>~/.ssh/config <<END
        Host cashflow_cloud
          HostName $SSH_HOST
          User $SSH_USER
          IdentityFile ~/.ssh/deploy.key
          StrictHostKeyChecking no
      END
```

En el mismo artículo explica la parte donde se configuran esos *secrets* en el repo.

> [!Note]
> Los secrets se configuran en repo -> Settings -> Security -> Secrets and Variables -> Actions. Cuando se llegue ahí se crean los secrets en "Repository secrets".
> .
> También se pueden crear con el GitHub CLI. [Ver](https://cli.github.com/manual/gh_secret_set)

Las variables:

- `secrets.PRIVATE_KEY`: es la llave ssh privada generada para el servidor
    - se le hace `cat` y sale el valor a copiar
- `secrets.KNOWN_HOSTS`: es el contenido del archivo en `~/.ssh/known_hosts` en el computador local
    - se puede copiar eso mismo siguiendo los pasos de abajo.

> Para configurar el valor para `known_hosts` la más fácil es hacer ssh desde el computador local y tomar el texto que haya en ese mismo archivo local. Copiarlo y pegarlo en la configuración del secret.

- `secrets.TARGET_HOST`: la IP del servidor en Linode.
- `secrets.TARGET_USER`: el nombre de usuario, en este caso ubuntu.

Luego, para acceder al servidor y ejecutar el comando es como si se hiciera desde el computador local:
```bash
- name: Log in server and run script
  run: |
    ssh cashflow_cloud 'bash ~/cashflow/deployments/api-release/scripts/deploy_api.sh'
```

El nombre `cashflow_cloud` lo definí en la llave Host en el paso anterior. Esta parte la vi fue en este [otro artículo](https://dev.to/martinandersongraham/using-github-actions-to-deploy-updates-to-my-vps-40el).

