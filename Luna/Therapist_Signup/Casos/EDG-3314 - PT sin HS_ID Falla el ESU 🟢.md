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

# Posible Causa 🚨

Esto del signup normal falló porque pasa de manera sincrona.
```ruby
Credentialing::HubspotContactService.new(therapist: therapist).create
```

Para dos casos ya. Ambos registros creados el mismo día del reporte.

## La Causa

Archivaron el día 26 de Enero la propiedad `employer_name`. Esa se usa en la clase de HubSpot y por eso explota al no estar.

## Afectados

26 de Enero:
- `ljobrien@bellsouth.net` - 🟢
- `rtknebel@gmail.com` - 🟢

27 de Enero:
- `rkowalin@gmail.com` - 🟢
- `tbartamian1@gmail.com` - 🟢
- `bjm11233211@gmail.com` - Es el mismo de abajo
- `brianjohnmahalaris@gmail.com` - 🟢


### Solución los de Enero 26 🟢

Actualizar el therapist en la base de datos para que tenga su HubSpot ID.

Therapist ljobrien:
```ruby
t = Credentialing::Therapist.find_by(email: "ljobrien@bellsouth.net")
t.update_column_with_audit(:hubspot_id, 193986940447, audit_comment: "Missing hubspot_id after initial signup")
```

Therapist rtknebel:
```ruby
t = Credentialing::Therapist.find_by(email: "rtknebel@gmail.com")
t.update_column_with_audit(:hubspot_id, 45258701, audit_comment: "Missing hubspot_id after initial signup")
```

### Solución los de Enero 27 🟢

Therapist rkowalin:
```ruby
t1 = Credentialing::Therapist.find_by(email: "rkowalin@gmail.com")
t1.update_column_with_audit(:hubspot_id, 149866301, audit_comment: "Missing hubspot_id after initial signup")
```

Therapist tbartamian1:
```ruby
t2 = Credentialing::Therapist.find_by(email: "tbartamian1@gmail.com")
t2.update_column_with_audit(:hubspot_id, 144356774433, audit_comment: "Missing hubspot_id after initial signup")
```


Therapist brianjohnmahalaris:
```ruby
t3 = Credentialing::Therapist.find_by(email: "brianjohnmahalaris@gmail.com")
t3.update_column_with_audit(:hubspot_id, 78460955, audit_comment: "Missing hubspot_id after initial signup")
```