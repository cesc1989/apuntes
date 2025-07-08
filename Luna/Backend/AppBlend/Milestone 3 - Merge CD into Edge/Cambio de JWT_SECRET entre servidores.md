# Cambio de `JWT_SECRET` impidió decodificar tokens

El `JWT_SECRET` del antiguo servidor de physician portal tenía un valor diferente al que está en `backend`. Esto causó un problema al tratar de abrir enlaces vencidos una vez se completó la migración a backend.

Como el `JWT_SECRET` que se usó para codificar el token es diferente al que lo decodifica, se lanza una excepción `JWT::VerificationError`. Para esos casos la clase que hace la captura de las excepciones devolvía un error en la respuesta JSON.

Devolvía:
```json
name: "auth_error",
message: "Something went wrong when authenticating."
```

Sin embargo, para otro link más reciente y también vencido, sí se mostraba la UI. Esto pasaba porque para ese link el JSON de la respuesta era:
```json
name: "expired",
message: "This authentication session has expired."
```

En el valor de `name` está la diferencia. El cliente frontend espera el valor `expired` para mostrar al usuario la UI con el botón "Request new link".

![[request new cd link.png]]

Al cambiar la captura de esa excepción en particular y devolver el valor esperado en `name` se corrigió el error para los usuarios que guardaron links anteriores.