# EDG-3204 - Actualización del Credentialing object incorrecto luego de 2do Sign Up

Etiquetas: #luna_help_desk 

El caso se resume así:
> The therapist initially signed a contract for Illinois in error and subsequently signed a corrected contract for Florida. When the therapist completed their Florida Attestation Form as part of their Florida onboarding, the submission updated fields on the _Illinois credentialing_ object rather than the _Florida credentialing_ object.
>
> _**This issue appears to occur in relocation scenarios where a therapist has multiple credentialing objects. There may be cases where the backend is referencing an incorrect credentialing object during form submission.**_

El último párrafo es precisamente lo que pasa.

# Datos

Estos son los datos del contacto ahora mismo:
- Correo: danielleg1105
- Contact ID: 190930253327
- AF URL: `attestation/progress/64a80c76-6c99-4633-bee2-16f01db4ab0d`

Están en la forma esperada cuando comparo con el registro en la base de datos.

Registro en la bd:
- ID: `64a80c76-6c99-4633-bee2-16f01db4ab0d`
- Correo: danielleg1105
- HS ID: 141970001
- Crede AA ID: *37142432423* ❌
	- apunta al Credentialing incorrecto

> [!Note]
> El HS ID es diferente pero porque el contacto pasó por varios merges. Al enviar peticiones se actualiza el Contacto actual.

Aquí vienen las diferencias y problemas.

Credentialing de Illinois:
- ID: *37142432423*
- Label: Inactive

Credentialing de Florida:
- ID: 43321731866
- Label: Processing for Move

# Sospechas 👀

## Sospecha #1

No se hace más syncs de Custom Object IDs porque el guard en las funciones de sincronización.

Esto:
```ruby
return if therapist.credentialing_active_attested_id.present?
```

En la función `maybe_schedule_credentialing_object_ids_sync`. Ahora mismo el registro en la BD ya tiene un valor en `credentialing_active_attested_id` por lo tanto no se hace el resync.

# Soluciones 🚧

## Alternativa #1

Asegurar que en todas las secciones del CA/AF se esté bajando el ID de cada objeto.