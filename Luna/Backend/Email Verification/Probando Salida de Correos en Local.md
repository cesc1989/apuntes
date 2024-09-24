# Probar Salida de Correos en Local

En este documento listo el proceso para probar la salida de correos en Edge. Así no tener que esperar a que salga en alfa.

## Mailcatcher

Instala mailcatcher:

```
gem install mailcatcher --no-document
```

Si falla, ver [[Apuntes_Rails_-_Edge_Parte_1#Error al intentar instalar mailcatcher]]

## Usando Mailcatcher

Corre el servidor con:
```
mailcatcher

Starting MailCatcher v0.10.0
==> smtp://127.0.0.1:1025
==> http://127.0.0.1:1080
*** MailCatcher runs as a daemon by default. Go to the web interface to quit.
```

Y accede en `http://127.0.0.1:1080`.