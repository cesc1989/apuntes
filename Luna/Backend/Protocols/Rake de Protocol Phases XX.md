# Apuntes sobre Protocol Phases rake task

Esta es una rake task que se corre cada tanto tiempo (muy esporádicamente) cuando el equipo de Physician Management pide cargar nuevos Protocols al sistema.

> Se corre el script porque la UI para esto sería complicada de hacer y se usaría muy poco.

# Datos Clave

- La rake está en `lib/tasks/protocols.rake`
- Los seeds están en `db/seeds/protocols`
	- Los seeds son archivos YAML.
- Todos los nuevos protocolos se copian a la carpeta `db/seeds/protocols`
	- Quien pide cargarlos al sistema envía un zip que contiene todos los protocolos existentes + los nuevos.
	- **Aquí hay que copiar y pegar todo. No vale la pena buscar cuáles no están. Copia y pega todo.**
- A veces también piden cargar Pathways. Esto es un archivo YAML también.
	- Este archivo se copia en `db/seeds/`

El resumen de las instrucciones del runbook en Notion es:

- Cambia el nombre de la rake para que el consecutivo sea un número mayor al actual.

```diff
- task :phase_24, %i[live_mode] => :environment do |_task, args|
+ task :phase_25, %i[live_mode] => :environment do |_task, args|
```

- Cambia el consecutivo y fecha en la descripción del rake.

```diff
- desc "Phase 24 (Aug 1, 2024) protocol load"
+ desc "Phase 25 (Feb 5, 2025) protocol load"
```

- Limpia cualquier lógica de las funciones:
	- `fix_incorrect_pathway_assignments`
	- `split_and_reword_callouts`
	- `fix_callout_text_inconsistencies`
	- `fix_escalation_question_text_inconsistencies`
	- `rename_pathways`
	- `migrate_existing`

La lógica presente en esas funciones muy seguramente aplicaría solo a la ejecución anterior.

- Revisa que los archivos YAML están bien formateados. No hay espacios de sobra.
- Ejecuta varias veces el script en una copia local de producción

> [!Note]
> Como no tengo acceso, debo pedir ayuda a otro dev que sí lo tenga.

# Ejecución

La rake la puedo correr en modo seguro así:
```bash
# bundle exec rake protocols:phase_<x+1>

bundle exec rake protocols:phase_26
```

Y en modo producción hay que correrla así:
```bash
bundle exec rake protocols:phase_26\[true\]
```


# Algunos errores y detalles de las tablas

En mi primer vez que trabajé en esto descubrí varias cosas sobre cómo está definido el modelo alrededor de Protocol Phases. Describo algunas de esas a continuación.

## Error de violación de índice de exclusión

Este error:
```bash
Caused by:
PG::ExclusionViolation: ERROR:  conflicting key value violates exclusion constraint "protocol_phases_802620237"
DETAIL:  Key (relative_period, protocol_id)=([42,), 81024eba-c684-48e0-bbff-d07c5cac38c0) conflicts with existing key (relative_period, protocol_id)=([38,49), 81024eba-c684-48e0-bbff-d07c5cac38c0).
/Users/ellamosi/Workspace/backend/app/seeders/protocol_seeder.rb:56:in `seed'
```

Pasó porque en un protocolo nuevo se definía esto:
```yml
phase_periods: ["1:10", "11:23", "24:37", "38:48", "49:"]
```

El último rango está mal definido. Los rangos en `phase_periods` deben ser excluyentes. El último valor no puede coincidir con el primer valor del siguiente rango.

Este índice de exclusión lo veo en el schema:
```ruby
t.exclusion_constraint "relative_period WITH &&, protocol_id WITH =", using: :gist, name: "protocol_phases_802620237"
```

También veo que el campo `relative_period` se de tipo `int4range`. ¿Qué es eso?

## Error de Protocol still missing documents

Sale así:
```bash
Protocols still missing documents: ["54fd92d5-a6f1-47f5-84d2-72168b968a18", "75d71a38-30fe-4a49-9ec8-44e1321a16df"]
```

Solución: en este caso hay que pedirle a los del equipo de Clinical que tienen que cargar el archivo PDF del Protocol en Luxe. O revisar si le pusieron el nombre mal.

## Error de Protocol Mismatch via [Database/Seeds]

Cuando corrí en local protocols phase 26 me dio este error en ambas versiones del comando:
```ruby
Protocol ID mismatch: Extras via database: <Set: {\"ac39e49d-7fa0-468b-8f9a-be904dac8415\", \"6298c2b5-bdbb-4ae7-830a-ca9a438b19e7\", \"ad43db53-2476-4934-8027-215d587d1088\"}>
```

En el rake está controlador por la función `validate_protocol_numbers` que es la última en ejecutarse. La función cuenta los seeds en `db/seeds/protocols/` y los que hay en base de datos.

Cuando hay más extras en base de datos el mensaje es: `Protocol ID mismatch: Extras via database`. Cuando hay más extras en los seeds el mensaje es: `Protocol ID mismatch: Extras via seeds`

En este caso los IDs de los protocols son "Non Production". Creo que es porque ejecuté la rake pasada (phase 25) en mi local y en alpha.