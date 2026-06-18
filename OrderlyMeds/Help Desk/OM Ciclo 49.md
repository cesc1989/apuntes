# Ciclo 49 de OrderlyMeds Support

## Caso OM-9481 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("paigekaye@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9350 - Reset Check In CX en Prospect 🟡

Etiquetas: #om_cx_in_prospect

Dicen esto:
> Customer wants to order, her acct was merged previously. Please reset link so cx can order thank you

Pero cuando voy al perfil del cliente en Success sale que estado es "Prospect"

### ¿Qué significa Prospect?

Según Jay:
> means that cx haven't placed an order yet

### CX debe completar el HealthScreening

Para resolver el cliente debe completar esta parte (hacer una orden). Se debe ubicar el enlace que lleva a esta parte en Ontraport. Está en la sección "Activity" del perfil del CX. Buscar el campo "ShortUrl Healthscreening".

Copiar e indicar a CS que deben pedirle al CX que complete esa etapa primero.

## Caso OM-9473 - Oops Error 🟢

Etiquetas: #om_oops_error 

Todos:

- [x] Revisar el account
- [x] Actualizar el campo `workos_user_nk`
- [x] Sincronizar salesforce

## Caso OM-9419 - errr??

xxx

## Caso OM-9426 - Stuck in Submitted

Seguí los pasos del caso [[OM Ciclo 48#Caso OM-9337 - Stuck in submitted 🚨]]:

- [x] Cambiar estado a `needs_resubmission`
- [x] Hacer resubmit en Ontraport
- [x] Comprobar nuevo CareValidate::Request creado
- [x] Comprobar Script pasó a Pharmacy Selected
- [x] Indicar a CS que el script fue resubmiteado

Mensaje para CS:
```
Script resubmitted. Please check it out.
```