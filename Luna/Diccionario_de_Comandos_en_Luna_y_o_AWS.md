# Diccionario de Comandos en Luna y/o AWS

# Servicios en AWS

## SSO Login for credentials Access

```bash
aws sso login --profile [PROFILE] # alpha, beta, omega

# example
aws sso login --profile alpha
```


## Download or upload files to a S3 Bucket

See [[Copiando_Archivos_a_desde_S3_mediante_CLI]]

## Upload Therapist Signup Agreement PDF Files

See [[Copiando_Archivos_a_desde_S3_mediante_CLI]]

## List files in S3 bucket per conditions

See [[How_to_list_files_per_condition_in_S3_Bucket]]

# Patient Forms

## Create dev DB

```bash
luna rds create-dev-db -p alpha -d patient-self-report
```

## Find an incomplete Intake Form

```bash
# 1st log into a container using Luna CLI
# Inside the container
bundle exec rails c

# Inside rails console
Form.onboarding.uncompleted.order("RANDOM()").first.link
```

## Find an incomplete Progress Form

```bash
# 1st log into a container using Luna CLI
# Inside the container
bundle exec rails c

# Inside rails console
Form.progress.uncompleted.order("RANDOM()").first.link
```

# Edge

## Create dev DB

```bash
luna rds create-dev-db -p alpha -d edge
```

# Marketplace

## Create dev DB

```bash
luna rds create-dev-db -p alpha -d marketplace
```

## When connection to Dev DB time outs

Need to whitelist the public IP:
```bash
luna rds whitelist-ip-for-dev-db --profile alpha
```

# Usar string de conexión de Postgres

Una string:
```bash
postgres://luna_api:U9fc8Em1tCmGkdGjhBAkuiL7W85*hB@dev-francisco-alpha-edge-12-15t15-44.ctvhkhbiykgu.us-west-2.rds.amazonaws.com:5432/luna_api_staging
```

Se traduce en

- username:  `luna_api`
- password:  `U9fc8Em1tCmGkdGjhBAkuiL7W85*hB`
    - después de los dos puntos y antes del arroba
- host name: `dev-francisco-alpha-edge-12-15t15-44.ctvhkhbiykgu.us-west-2.rds.amazonaws.com`
    - después de los dos puntos luego del pass y hasta el `.com`
- port: `5432`
    - casi siempre es este
- database name: `luna_api_staging`

## En Edge / Backend

El proyecto Edge/Luxe/Backend tiene la env `DEV_DATABASE_URL`. Ahí podemos pegar la string completa:
```bash
DEV_DATABASE_URL="postgres://luna_api:U9fc8Em1tCmGkdGjhBAkuiL7W85*hB@dev-francisco-alpha-edge-12-15t15-44.ctvhkhbiykgu.us-west-2.rds.amazonaws.com:5432/luna_api_staging"
```


## En DBeaver

Hay que copiar los valores uno a uno la primera vez. La segunda ocasión, para una misma conexión, solo reemplazamos: “host name” y “password”.

![[dbevaer.setup.png]]


