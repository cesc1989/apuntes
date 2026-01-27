# EDG-3314 Therapist sin hubspot_id al hacer 2do registro

Etiquetas: #luna_help_desk 

## Reporte

Un therapist intenta registrarse pero le da el error:
```
Email "x" is already in use.
```

Se me hace raro porque el Extended Sign Up se hizo para saltarse eso. ¿Qué pasa?

## Contexto

En el Extended Sign Up hay varios chequeos para decidir si se hace un "nuevo" signup si el email está en uso.

Ver [[Extended Sign Up]]

Para lograrlo se comprueba que el email esté en uso. Es la condición clave. Eso quiere decir que la persona está intentando registrarse de nuevo.

Luego de eso se revisa que el therapist tenga un hubspot_id en la base de datos. Si no lo tiene, se detiene el proceso.

Aquí es donde falla el proceso. El therapist no tiene el hubspot_id y por eso el último error que se devuelve es el del email tomado.

## Posible Causa

Esto del signup normal falló porque pasa de manera sincrona.
```ruby
Credentialing::HubspotContactService.new(therapist: therapist).create
```

## Solución

Actualizar el therapist en la base de datos para que tenga su HubSpot ID.

```ruby
t = Credentialing::Therapist.find_by(email: "ljobrien@bellsouth.net")
t.update_column_with_audit(:hubspot_id, 193986940447, audit_comment: "Missing hubspot_id after initial signup")
```