# Apuntes Ciclo 12 - Seguridad - Coshi Notes
Luego de revisar el subdominio en Shodan

# Configuraciones de Seguridad
## Desactivar para que passenger no muestre la versión

Recursos:

- Docs → https://www.phusionpassenger.com/docs/references/config_reference/nginx/#passenger_show_version_in_header
- Explicación sobre el [bloque de contexto](https://stackoverflow.com/a/48446892/1407371) donde debe ir cada configuración


## Forzar SSL en Rails

Activar la opción

    force_ssl = true

Tiene varias [características](https://stackoverflow.com/questions/15676596/what-does-force-ssl-do-in-rails) de seguridad importantes.

Según este artículo, para activar el force_ssl hay que tener el certificado ssl configurado en nginx. Ya todo eso lo tenía listo.

https://www.matthewhoelter.com/2020/02/19/how-to-setup-force-ssl-on-nginx-and-lets-encrypt.html


Solo fue activar y se pudo ver que la cookie de sesión estaba marcada como `secure = true`.


# Replicando error de Devise 4.9 + Rails 7.1.0

Hice un [repo de prueba](https://github.com/cesc1989/devise_rails_session_repro) para tratar de encontrar el error que había tenido pero no di con la causa.

Tarjeta en Trello → https://trello.com/c/RMEW79PK


