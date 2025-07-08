# 003 - Dos Credentialing Objects con label Active

> Found a new issue with moving therapist scenario (new state). When I sign a new contract, it creates a duplicate credentialing object using the label 'Active'.

Contacto: [https://app.hubspot.com/contacts/4634981/record/0-1/134628683368](https://app.hubspot.com/contacts/4634981/record/0-1/134628683368)

Cred 1 (correcto): [30278823850](https://app.hubspot.com/contacts/4634981/record/2-31374266/30278823850)

Cred 2 (duplicado): [30272441773](https://app.hubspot.com/contacts/4634981/record/2-31374266/30272441773)

## Datos de cada objeto

El Credentialing 1 fue creado el **Jul 3, 2025 at 8:30 PM GMT-5**.

El Credentialing 2 fue creado el **Jul 4, 2025 at 2:04 AM GMT-5**.

---

El Credentialing 1 tiene varias propiedades con valores. Esto significa que era el "Active Attested" anterior.

El Credentialing 2 carece de valores en muchísimas propiedades.


# Flujos que crean Credentialing objects

## Primer Credentialing

El primer Credentialing se crea cuando un Therapist (que no existe en el sistema) completa el Sign Up form. **Este objeto es creado automáticamente por HubSpot**.

El ID de este objeto Credentialing es guardado en el campo `credentialing_active_attested_id` porque inicialmente tiene la label "Active Attested".

## Segundo Credentialing

**El segundo Credentialing mediante un llamado a la API de HubSpot**. Esto ocurre cuando se completa el Extended Sign Up: un mismo therapist completa el Sign Up form nuevamente.

En este flujo se crea un nuevo Credentialing con label "Active Attested". Mientras tanto, al Credentialing initial se le cambia la label de "Active Attested" a "Active".

El fujo, en términos de uso de la API de HubSpot, es así:

- Crea segundo Credentialing (aún no se asocia, solo existe)
- Si el campo `credentialing_active_attested_id` tiene un valor:
	- Agrega la label "Active" al 1er Credentialing
	- Quita la label "Active Attested" al 1er Credentialing
- Asocia el 2do Credentialing con el Contact mediante la label "Active Attested"

Al final el flujo, los objetos Credentialing deben quedar así:

- 1er Credentialing: Active label
- 2do Credentialing: Active Attested

# Posibles estados de labels: ¿se puede un Active - Active?

> [!Important]
> Pedí a Claude revisar los posibles flujos y no se encuentra un resultado que produzca dos objetos con la misma label "Active"

tbc


# Revisar historial de labels de una asociación

HubSpot no brinda una forma de saber en qué fecha una label fue asignada a un objeto. Si hubiera esto, podría revisar las fechas en que el objeto original obtuvo su label "Active". Lo mismo para el duplicado.

En la UI se puede llegar a ver el [historial de las asociaciones](https://knowledge.hubspot.com/records/associate-records#view-a-record-s-association-history) pero no arroja mucha información que ayude.

![[003.association.history.png]]