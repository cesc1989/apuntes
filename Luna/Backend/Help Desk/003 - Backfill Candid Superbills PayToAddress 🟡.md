# 003 - Backfill de Candid Superbills PayToAddress

Etiquetas: #luna_help_desk 

Report:
> The Pay_to_address has been updated for all of the addresses to separate addresses for each location (a different PO Box in Atlanta for each)

Es una rake que existía y luego se borró. Restauré todo con Claude. No entendía muy bien de qué va así que le pregunté y dijo:

> The `backfill_pay_to_address` rake task (lines 55-74) is designed to update Candid encounters for Luna Direct clinics that have PO Box addresses.
>
> **Purpose**: Updates the `pay_to_address` field for existing Candid encounters where clinics have PO Box addresses.
>
> What it does:
> 1. Finds appointments that meet these criteria:
>	- Have associated Candid encounters
>	- Scheduled between Jan 1, 2023 and Sep 15, 2025
>	- Belong to clinics where `pay_to_address` starts with "PO Box"
> 2. Schedules background jobs: For each matching appointment, it queues a `Candid::Egress::PayToAddressWorker` job with a 2-second delay between jobs to avoid overwhelming the system.
> 3. Provides progress info: Shows total count and estimated completion time.



## Luna Direct Clinics - Luna Care Practice

La descripción de la rake dice:
> Backfill pay_to_address field for Luna Direct clinics with PO Box addresses

Meredith también me indicó que debía incluir esas clinics solamente:
> you're missing a where episode.clinic = the luna clinic in this query

Claudio encontró que:

- Luna Care is indeed a practice with many clinics
	- El archivo `db/seeds/practices/luna_care.yml` lista todas sus clinics
	- Dicho archivo fue actualizado en este https://github.com/lunacare/backend/pull/12344
	- Lo que está en el archivo coincide con que estas son Clinics de Luna Care practice

Ejemplo en alpha DB:
```ruby
Practice.find_by(key: 'luna_care')
=> #<Practice:0x0000000127f1e7d0
 id: "3bbc299b-b110-49f6-afc5-23523b08d67d",
 name: "Luna Care",
 referral_code: "LUNA",
 created_at: Tue, 15 Jun 2021 12:36:51.462998000 PDT -07:00,
 updated_at: Wed, 30 Jul 2025 08:04:23.577033000 PDT -07:00,
 parent_id: nil,
 relationship_to_company: "self_",
 key: "luna_care",
 notes_for_record_viewer: "",
 notes_for_booking_agent: "">
 
 Practice.find_by(key: 'luna_care').clinics.first
 => #<Clinic:0x0000000139d86870
 id: "0138126b-5c0b-4595-a696-23925701da0d",
 practice_id: "3bbc299b-b110-49f6-afc5-23523b08d67d",
 region_id: 241,
 tin_id: 95,
 provider_name: "Luna Care, Inc.",
 national_provider_identifier: "1518576164",
 key: "luna_care_milwaukee",
 clinicient_routing_key: "36",
 created_at: Thu, 29 Jul 2021 20:57:56.745741000 PDT -07:00,
 updated_at: Wed, 30 Jul 2025 08:04:25.174260000 PDT -07:00,
 routing_strategy: "standard",
 benefit_level: "regular",
 fax_number: nil,
 fax_every_visit: false,
 billing_name: "Wisconsin Luna Care Physical Therapy, LLC",
 npi_id: 22,
 billing_address: "525 Junction Rd",
 billing_address_2: "Suite 6500",
 billing_address_city: "Madison",
 billing_address_state: "WI",
 billing_address_zip_code: "53717-2153",
 pay_to_address: "PO Box 290609",
 pay_to_address_2: nil,
 pay_to_address_city: "Nashville",
 pay_to_address_state: "TN",
 pay_to_address_zip_code: "37229-0609",
 state_id: "082528c0-8bb4-4d44-84d2-6c3b4af79ac9",
 billing_type_id: "0503d240-4ac0-4452-8d4d-ca87c8566c9f",
 taxonomy_code: "193200000X",
 notes_for_record_viewer: "",
 notes_for_booking_agent: "">
```


# Revisión de Estado y Actualización de rake

La forma en que este proceso corre es muy lento porque tiene que actualizarse un Candid Encounter a la vez. No hay API batch a la fecha. Confiaba en que con el pasar de los días se iba a ir viendo reflejado el update pero Christie a comentado que no es así.

Entonces me puse con Claudio a:

- actualizar la rake inicial para que
	- reciba un rango de fechas
	- eliminar el paso dry run
		- con la fecha de cierre se puede probar con un conjunto de datos más pequeños
- crear una nueva rake para verificar si los encounters fueron actualizados
	- recibe también un rango de fechas para controlar el conjunto de datos
	- imprime al final una lista de IDs para verificar

Estas son las rakes en general

## Rakes actualizadas

### Backfill

Para correr el backfill en todo el conjunto de años:
```bash
bundle exec rake candid:backfill_pay_to_address
```

Para correr el backfill con rango personalizado:
```bash
bundle exec rake candid:backfill_pay_to_address["2023-01-01","2023-12-31"]
```

### Verificación

Verificación total de todos los años:
```bash
bundle exec rake candid:verify_pay_to_address_updates
```

Verificación con rango personalizado:
```bash
bundle exec rake candid:verify_pay_to_address_updates["2023-01-01","2023-12-31"]
```
