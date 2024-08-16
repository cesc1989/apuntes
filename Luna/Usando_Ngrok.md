# Usando Ngrok

Para poder revisar y trabajar en la solución de XXX tocó usar Ngrok de una forma particular.

El error solo era visible al abrir un formulario mediante un navegado *in-app* de aplicaciones móviles. Para lograr reproducir esto en ambiente local, era necesario poder compartir un enlace al formulario mediante Slack y abrirlo en el celular.

Como al ambiente local se ejecuta en el computador, al abrir el enlace en el celular el formulario no abriría porque no tiene acceso al localhost.

> Creo que esto se podría haber solucionado mediante acceso al computador usando la IP de la red local.

Entonces, entró Ngrok. Lo que se trató de hacer fue hacer el API y el cliente frontend disponible en internet pero ejecutando el entorno local.

## Pasos Seguidos

Para ejecutar esta misión con éxito se hizo lo siguiente.

### Backend

**1)** Lanzar un proceso de Ngrok: `~/ngrok/ngrok http 3000`

**2)** La URL arrojada se agregó en `config/environments/development/`

```ruby
 Rails.application.configure do
  # Since Rails 6.
  # Read about at https://blog.bigbinary.com/2019/11/05/rails-6-adds-guard-against-dns-rebinding-attacks.html
  config.hosts << 'api.lvh.me'
  config.hosts << '7b0ba3b3f641.ngrok.io'
end
```

> También tocó quitar la restricción de subdominio del API.

**3)** Levantar servidor de rails `rails server` normalmente

### Frontend

**1)** Cambiar URL base del API por la que se generó con Ngrok en backend. En el archivo `src/api.js`

**2)** Levantar servidor NPM: `npm start`

**3)** Levantar Ngrok: `ngrok http 8080`

Ya con lo anterior era posible abrir un formulario (datos en backend) trayendo los datos del API a través de Ngrok(túnel creado por mí) que eran reflejados en el cliente frontend en el túnel creado por Ngrok en el computador de Nicolas.

## ¿Se puede más fácil?

Al parecer una forma de poder levantar todo esto desde un mismo computador (si no se paga Ngrok solo se puede un comando a la vez) sería usando una configuración de túneles como se describe en [esta parte de la documentación](https://ngrok.com/docs#multiple-tunnels).

Habría que usar un archivo de configuración que Ngrok leería:
```yaml
 authtoken: TOKEN
    tunnels:
      api:
        addr: 3000
        proto: http
        inspect: true
      client:
        addr: 8080
        proto: http
```

Y lanzar el comando de Ngrok así:

```bash
$ ~/ngrok/ngrok start -config ~/Downloads/api_and_client.yml api client
```

Realizar los pasos **2 y 3** descritos en la sección “Backend” arriba. Y de la sección “Frontend” solo ejecutar los pasos **1 y 2**.
