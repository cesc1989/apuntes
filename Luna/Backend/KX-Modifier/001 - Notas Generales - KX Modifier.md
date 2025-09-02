# Notas Generales - KX Modifier

Sistemas involucradas:
- 3rd party systems
- Luxe
- Mobile Apps

Aplicaciones involucradas:
- Backend
- Therapist App

## Problema

Si el **paciente** está ==a punto de gastar más de 2000 dólares en Medicare==, hay que pedir confirmación si se necesita más tratamiento.

> [!Note]
> De no hacerlo podría pasar que:
>
> - Los reclamos sean rechazados.
> - Luna podría incumplir normativas.

> [!Note]
> Esto se hace a partir de los 2000 dólares como *medida proactiva* por parte de Luna. El límite real son 2330 dólares.

## Solución

Hay que monitorear el gasto en Medicare.

Datos:
- Límite real es de 2330 dólares.
- ==Cada visita de un Care Plan, luego de exceder el límite, necesita el código KX para poderse facturar==.
- Usar el código KX indica que el terapeuta o el doctor *confirman que se necesita más tratamiento*.
- Solo se necesita hacer ==una vez al año por cada Care Plan==.

En Luxe, se habrá un checkbox para mostrar si el paciente excede o no el límite.

"Patient Exceeds Limit [ ]"

## Condiciones para mostrar el mensaje de confirmación

> Este mensaje se muestra en la aplicación móvil de Therapists.

Se mostrará si el paciente cumple **cualquiera** de estas condiciones:

- Exceder el límite de 2000 dólares de Luna
- Tuviera un **reclamo denegado por código CO-119 claim_adjustment_reason_codes** // Candid
- Empezó ==otro Care Plan con Luna Y ya excedió el límite== de Medicare

## Acciones Internas

Cosas que debe hacer backend o la app móvil.

Si el therapist dice que "Yes"

- Todos los ==códigos CTP del Care Plan deberán tener agregado el código modificador KX==.
	- También debe agregarse al enviar un claim a Candid.

Si el therapist dice que "No"

- Reconfirmar el "No" (lo hace mobile)
- Se envía mensaje a Sendbird (¿mobile también?)

## Milestones

### Must Have

- Medical benefit spend debe guardarse al menos **cada dos semanas**.
- Mostrar ==checkbox en el perfil del Paciente que indica si excedió o no el límite==
	- "Medical necessity confirmed [ ]"
	- Se puede modificar por usuarios de Luxe
- Cada cambio debe poder *auditarse*
	- En el RFC, Alexis define un modelo nuevo que agrega el macro `audited`
- Billing team quiere el ==código KX agregado a toda entrada de facturación (billing) que vaya a Candid==.
- Al terminar cada año en curso, debe deschularse los checkboxes (campo en base de datos) de "medical Necessity confirmed" y "Patient Exceeds Limit"
	- El límite de Medicare se reinicia con cada año nuevo

**Lo que pide el equipo de Billing** con respecto a Candid

- As a Billing team member, I want ==KX Modifier codes added to all future billing entries for a patient in Candid== when they exceed their annual plan amount and care is still medically necessary.
- As a Billing team member, I want ==KX Modifier codes attached to all denied claims== when ongoing care is deemed medically necessary.