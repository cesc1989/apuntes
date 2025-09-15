# 002 - Respuesta 422 del endpoint v1/sessions

Etiquetas: #luna_help_desk 

Primero que nada, este endpoint no tiene forma lógica de devolver un 422.

## Problema

Brandon notó que había varias grabaciones donde se devolvía esa respuesta. Para el día 4 de Agosto hubo 46 ocasiones.

Lo que muestra una de las grabaciones:

![[002.logrocket.recording.png]]

Este es el breadcrumb de esa grabación:

![[002.logrocket.breadcrumb.png]]

Datos:

- endpoint: `api-outcomes.getluna.com/v1/sessions`
- method: POST
- status: 422

Cabeceras de respuesta:

- x-frame-options: SAMEORIGIN

## Replica en Local con Claudio

Claudio no pudo replicar esta respuesta.

Based on my testing, I can conclude that the `SessionsController#create` action does NOT return 422 errors. Here's what my testing revealed:

**Test Results Summary:**

1. Valid token → 201 Created ✅
2. No Authorization header → 401 Unauthorized
3. Invalid/malformed token → 401 Unauthorized
4. Empty token → 401 Unauthorized
5. Bearer instead of Token → 401 Unauthorized
6. Invalid JWT signature → 401 Unauthorized
7. Valid token with JSON/form data → 201 Created ✅
8. Malformed JSON body → 400 Bad Request

**Why `SessionsController` Doesn't Return 422:**

The create action in `SessionsController` has a very simple flow:

1. Token validation → All auth failures return 401 Unauthorized (handled by `with_jwt_auth_errors_handled`)
2. Success case → Returns 201 Created with dashboard info
3. No parameter validation → The action doesn't validate or require any specific request parameters

The controller doesn't have any validation logic that would trigger a 422 (Unprocessable Entity) response. All error paths lead to either:

- 401 for authentication/authorization failures
- 500 for unexpected server errors
- 201 for success

The `SessionsController#create` action cannot return a 422 error under normal circumstances.

## Causa

Al migrar Clinical Dashboard a Edge el `ApplicationController` pasó a tener mucho más código que el que estaba en el repo independiente. Dicho controller define lo siguiente:

```ruby
protect_from_forgery with: :exception
skip_before_action :verify_authenticity_token, if: :verify_api
```

Y a pesar de que el controlador base de Clinical Dashboard está así:
```ruby
protect_from_forgery with: :null_session
```

no impide que la verificación del token de autenticidad se lleve a cabo causando el error 422.

Este es el error que encontré en los logs:
```bash
ActionController::InvalidAuthenticityToken (The browser returned a 'null' origin for a request with origin-based forgery protection turned on.
```

Queda la duda, ¿por qué en alpha no ocurre?

En todo caso pude replicarlo con el comando curl:
```bash
curl -I -X POST https://api2.alpha.getluna.com/v1/sessions \
→   -H "Content-Type: application/json" \
→   -H "Origin: null" \
→   -H "Authorization: Token eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJwaHlzaWNpYW4iLCJleHAiOjE3NTc5NTQ1MjcsImlhdCI6MTc1Nzk1NDAyNy4xODQ2NjUyLCJpc3MiOiJwaHlzaWNpYW4tcG9ydGFsLWxpbmtzLWdlbmVyYXRvciIsIm5iZiI6MTc1Nzk1NDAyNywic3ViIjoiN2U2ZmU3MjgtYTJmOC00ZTc1LTk5MzUtY2YyMzg0OTk5Mzg1IiwicHJvdmlkZXJfbmFtZSI6IkFhcm9uIFNhbHlhcG9uZ3NlIiwicHJvdmlkZXJfa2luZCI6InBoeXNpY2lhbiIsInByb3ZpZGVyX2lkIjoiN2U2ZmU3MjgtYTJmOC00ZTc1LTk5MzUtY2YyMzg0OTk5Mzg1IiwicG9ydGFsX3JlY2lwaWVudF9lbWFpbCI6ImZyYW5jaXNjby5xdWludGVybysyNUBpZGVhd2FyZS5jbyIsInByb3ZpZGVyX2NvZGUiOm51bGwsImRhc2hib2FyZF92ZXJzaW9uIjoidjMiLCJkYXNoYm9hcmRfaWQiOiJkMzUxZTMyOC04OTg2LTRjMGQtOWNlMy1jNDBjYTMwMTAxOWMifQ.2--znKt-ckoqYm0sLj8qau3KyL5iR_jH_yDGHVc13WA"
HTTP/2 422
```


## Solución

Se da al saltarse esta verificación en el controlador afectado.
```ruby
module ClinicalDashboard
  module Api
    class SessionsController < Api::BaseApiController
      skip_before_action :verify_authenticity_token
    end
  end
end
```

Si intento hacerlo en el controlador Base me da este error en el CI:
```
/usr/local/bundle/gems/activesupport-7.1.4/lib/active_support/callbacks.rb:835:in `block (2 levels) in skip_callback': Before process_action callback :verify_authenticity_token has not been defined (ArgumentError)
```

Por más que probé lo único que sirvió al final fue poner el skip en el controlador en específico.

## Pruebas al aplicar la solución

Prueba en local:
```bash
curl -I -X POST http://localhost:3000/v1/sessions \
→   -H "Content-Type: application/json" \
→   -H "Origin: null" \
→   -H "Authorization: Token eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJwaHlzaWNpYW4iLCJleHAiOjE3NTc5NTQ1MjcsImlhdCI6MTc1Nzk1NDAyNy4xODQ2NjUyLCJpc3MiOiJwaHlzaWNpYW4tcG9ydGFsLWxpbmtzLWdlbmVyYXRvciIsIm5iZiI6MTc1Nzk1NDAyNywic3ViIjoiN2U2ZmU3MjgtYTJmOC00ZTc1LTk5MzUtY2YyMzg0OTk5Mzg1IiwicHJvdmlkZXJfbmFtZSI6IkFhcm9uIFNhbHlhcG9uZ3NlIiwicHJvdmlkZXJfa2luZCI6InBoeXNpY2lhbiIsInByb3ZpZGVyX2lkIjoiN2U2ZmU3MjgtYTJmOC00ZTc1LTk5MzUtY2YyMzg0OTk5Mzg1IiwicG9ydGFsX3JlY2lwaWVudF9lbWFpbCI6ImZyYW5jaXNjby5xdWludGVybysyNUBpZGVhd2FyZS5jbyIsInByb3ZpZGVyX2NvZGUiOm51bGwsImRhc2hib2FyZF92ZXJzaW9uIjoidjMiLCJkYXNoYm9hcmRfaWQiOiJkMzUxZTMyOC04OTg2LTRjMGQtOWNlMy1jNDBjYTMwMTAxOWMifQ.2--znKt-ckoqYm0sLj8qau3KyL5iR_jH_yDGHVc13WA"
HTTP/1.1 401 Unauthorized
```

Prueba en alpha:
```bash
curl -I -X POST https://api2.alpha.getluna.com/v1/sessions \
→   -H "Content-Type: application/json" \
→   -H "Origin: null" \
→   -H "Authorization: Token eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiJwaHlzaWNpYW4iLCJleHAiOjE3NTc5NTQ1MjcsImlhdCI6MTc1Nzk1NDAyNy4xODQ2NjUyLCJpc3MiOiJwaHlzaWNpYW4tcG9ydGFsLWxpbmtzLWdlbmVyYXRvciIsIm5iZiI6MTc1Nzk1NDAyNywic3ViIjoiN2U2ZmU3MjgtYTJmOC00ZTc1LTk5MzUtY2YyMzg0OTk5Mzg1IiwicHJvdmlkZXJfbmFtZSI6IkFhcm9uIFNhbHlhcG9uZ3NlIiwicHJvdmlkZXJfa2luZCI6InBoeXNpY2lhbiIsInByb3ZpZGVyX2lkIjoiN2U2ZmU3MjgtYTJmOC00ZTc1LTk5MzUtY2YyMzg0OTk5Mzg1IiwicG9ydGFsX3JlY2lwaWVudF9lbWFpbCI6ImZyYW5jaXNjby5xdWludGVybysyNUBpZGVhd2FyZS5jbyIsInByb3ZpZGVyX2NvZGUiOm51bGwsImRhc2hib2FyZF92ZXJzaW9uIjoidjMiLCJkYXNoYm9hcmRfaWQiOiJkMzUxZTMyOC04OTg2LTRjMGQtOWNlMy1jNDBjYTMwMTAxOWMifQ.2--znKt-ckoqYm0sLj8qau3KyL5iR_jH_yDGHVc13WA"
HTTP/2 401
```

Prueba en omega:
```bash

```