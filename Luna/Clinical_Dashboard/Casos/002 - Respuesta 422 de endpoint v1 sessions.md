# 002 - Respuesta 422 del endpoint v1/sessions

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