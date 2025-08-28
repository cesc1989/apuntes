# Apuntes de Bliss - Parte 1

## Probando POW

Este workflow arranca con un paciente. Hay dos cosas clave para poder completarlo:

- Info de Pago
- Schedule Initial Visit

### Info de Pago

Hay que meter los datos de Stripe falsos:

TC: 4242 4242 4242 4242

Lo demás cualquier fecha en el futuro.

### Schedule Initial Visit para IMS

> [!Note]
> IMS: In-Memory Scheduler es la pestaña "Scheduler" de esta sección.


Para poder completar esta sección se necesita que carguen agendas de citas. En alpha, es posible que no cargue para todas las direcciones del paciente.

Un paciente con el que pude obtener agenda en esta sección es Hipolito Gislason.

Una región y dirección a usar es Chicago:

```
804 West Belden Avenue, Chicago, IL, 60614
```

> [!Important]
> La clave en esta sección es _no_ elegir un therapist en la lista de selección.