# Ciclo 49 de OrderlyMeds Support

> [!Info]
> Del Jueves 18 de Junio al Miércoles 1 de Julio.

## Caso OM-9481 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("paigekaye@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9350 - Reset Check In CX en Prospect 🟢

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

## Caso OM-9426 - Stuck in Submitted 🟢

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

## Caso OM-9489 - New MP 🟢

Etiquetas: #om_new_mp #om_checkin_reset

```ruby
new_member_period("amandafalco11@gmail.com")
```

Corrí comando y respondí al Linear con:
```
👋🏾 CX is ready for check-in.
```

## Caso OM-9400 - Remove CC de Salesforce 🟢

Etiquetas: #om_remove_credit_card

Pasos:

- Busca el `Account` del CX
- Luego lee las CCs que tenga adjuntas con `account.salesforce_account.card_payment_methods`
- Si solo es una, o a la que se necesite, la actualizas el campo `status` a `"Inactive"`

```ruby
account = Account.find("CHANGEME")

sfacc = account.salesforce_account
ccs = sfacc.card_payment_methods

ccs.first.update!(status: "Inactive")
```

## Caso OM-9347 - Script Error 🟢

Etiquetas: #om_script_error 

Cuando se revisa el Script en Ontraport el Outcome dice "Script Error". A pesar de hacer un resubmit sigue dando el mismo problema.

### Sin solución? Cancel & Refund ℹ️

> [!Info]
> Esto como que fue una situación dada por el paso al plan de 2 meses. CS tiene que hacer cancel & refund.

Este sería el caso para medicamentos de 3 meses. Fueron deshabilitados.

Mensaje para CS:
```
Currently, the 3-month option is no longer available. Therefore, a cancellation and full refund are required. The customer will then need to complete a new checkin and select either the 1-month or 2-month option.
```

## Caso OM-9396 - Invalid Drug ID - SF 🟢

Etiquetas: #om_invalid_drug_id

> [!Note]
> Este es lo mismo que el caso OM-9347. La inhabilitación del plan de 3 meses hace que este MP quede inválido. Se necesita hacer cancel & refund y nuevo MP.

Le pregunté a Fabian y me dijo que es lo mismo que el OM-9347. Dejé el mensaje en el hilo y moví el caso a "waiting on support".

### Verificación que está en la opción de tres meses

Para esto fui a Salesforce y revisé el Member Period. Pude ver en la sección "Med Picker Recommendation" que dice en "Product" "3-month Tirzepatide (High Dosage)". Ahí queda claro lo que me dijo Fabian.

## Caso OM-9426 - Stuck in Submitted 🟡

Seguí los pasos del caso [[OM Ciclo 48#Caso OM-9337 - Stuck in submitted 🚨]]:

- [x] Cambiar estado a `needs_resubmission`
- [ ] Hacer resubmit en Ontraport
- [ ] Comprobar nuevo CareValidate::Request creado
- [ ] Comprobar Script pasó a Pharmacy Selected
- [ ] Indicar a CS que el script fue resubmiteado

Mensaje para CS:
```
Script resubmitted. Please check it out.
```