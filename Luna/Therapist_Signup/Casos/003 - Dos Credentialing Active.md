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

## Logs

Revisé logs y encontré entradas para ambos IDs.

**Para el Credentialing correcto**
```
1751615198600	2025-07-04T07:46:38.600Z	  Therapist Update (0.9ms)  UPDATE "therapists" SET "updated_at" = $1, "credentialing_hubspot_id" = $2 WHERE "therapists"."id" = $3  [["updated_at", "2025-07-04 07:46:38.597697"], ["credentialing_hubspot_id", 30278823850], ["id", "f6c02957-0580-4afb-81fc-c29a4388c6ac"]]

1751612696846	2025-07-04T07:04:56.846Z	  Therapist Update (1.0ms)  UPDATE "therapists" SET "updated_at" = $1, "credentialing_hubspot_id" = $2 WHERE "therapists"."id" = $3  [["updated_at", "2025-07-04 07:04:56.843231"], ["credentialing_hubspot_id", 30278823850], ["id", "f6c02957-0580-4afb-81fc-c29a4388c6ac"]]

1751592700497	2025-07-04T01:31:40.497Z	  Therapist Update (0.7ms)  UPDATE "therapists" SET "updated_at" = $1, "credentialing_active_attested_id" = $2 WHERE "therapists"."id" = $3  [["updated_at", "2025-07-04 01:31:40.495455"], ["credentialing_active_attested_id", 30278823850], ["id", "f6c02957-0580-4afb-81fc-c29a4388c6ac"]]
```

**Para el incorrecto**
```
1751613764544	2025-07-04T07:22:44.544Z	  Therapist Update (1.5ms)  UPDATE "therapists" SET "updated_at" = $1, "credentialing_hubspot_id" = $2 WHERE "therapists"."id" = $3  [["updated_at", "2025-07-04 07:22:44.540820"], ["credentialing_hubspot_id", 30272441773], ["id", "f6c02957-0580-4afb-81fc-c29a4388c6ac"]]

1751612697460	2025-07-04T07:04:57.460Z	  Therapist Update (0.9ms)  UPDATE "therapists" SET "updated_at" = $1, "credentialing_active_attested_id" = $2 WHERE "therapists"."id" = $3  [["updated_at", "2025-07-04 07:04:57.456090"], ["credentialing_active_attested_id", 30272441773], ["id", "f6c02957-0580-4afb-81fc-c29a4388c6ac"]]
```

En [[Credentialing_Objects_State_Analysis]] está el momento a momento de lo que pasó. Claude sugiere que puede ser un problema de race condition. También sugiere otro montón de cosas.


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

La conclusión es que no hay forma directa en que se cree un objeto Credentialing con label Active.

El análisis completo está en [[Credentialing_Objects_State_Analysis]]

# Revisar historial de labels de una asociación

HubSpot no brinda una forma de saber en qué fecha una label fue asignada a un objeto. Si hubiera esto, podría revisar las fechas en que el objeto original obtuvo su label "Active". Lo mismo para el duplicado.

En la UI se puede llegar a ver el [historial de las asociaciones](https://knowledge.hubspot.com/records/associate-records#view-a-record-s-association-history) pero no arroja mucha información que ayude.

![[003.association.history.png]]

# Conclusión

> [!Note]
> Claudio sugirió varias cosas. Si logro replicarlo en alpha, las expondré a Brandon.

## 1era replica

Pude replicar el caso en un Contacto que ya tenía dos objetos Credentialing:

- Active Attested
- Active

Cuando hice un nuevo sign up se agregó un tercer objeto Credentialing y quedó todo así:

- Active Attested (nuevo Credentialing)
- Active (anterior AA)
- Active (1er active)

