# NEX-3377 Better fix for Medicare Requirements Answers generation

Etiquetas: #luna_help_desk 

## Contexto

De nuevo siguen los reportes. Por alguna razón los answers se crean muy después de lo esperado y cuando el therapist completa el CA no se registra la respuesta.

Therapists afectados:

- `9f4402af-3331-46b9-b9b7-748b6009c0e8`
- `91a50fb9-ab5d-4689-881d-32e8eedf9dd4`

## Propuestas

### Crear answers `no` por defecto ❌

> [!Warning]
> Este de momento no lo aplique porque entonces nada más al cargar el form la sección queda marcada como respondida por el frontend.

Cambiar la generación de las answers para que deje de saltar validaciones y que vayan con el `content` en "No". Además que haga todo en una sola consulta.

### Validar que las answers hayan sido respondidas antes del Submit 🟡

No había validación para esto en backend. Agregar las validaciones correspondientes para que si alguna answer no fue respondida con valores Yes/No, se marque el Medicare Req. como inválido.