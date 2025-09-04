# Apuntes Generales de Protocol Phases

Etiquetas: #luna_help_desk

# Protocol y su respectivo Document

¿Por qué es esto necesario?

## Relación entre Protocol y Document

Para crear un Protocol el Document es opcional:
```ruby
# Physician protocol definition.
class Protocol < ApplicationRecord
  # Definition
  belongs_to :document, optional: true
end
```

También se refleja en este scope:
```ruby
scope :configured, -> { where.not(document_id: nil) }
```

Y en la creación del mismo en el `ProtocolSeeder`:
```ruby
def protocol_data
	protocols_hash.map do |protocol_hash|
		# document uploading is handled outside of the seeder
		Protocol.new(
			id: protocol_hash.fetch("id"),
			name: protocol_hash.fetch("name"),
			version: protocol_hash["version"] || 1,
			parent_id: protocol_hash["parent"],
			phase_measure: protocol_hash["phase_measure"] || "surgery_date_offset_in_days"
		)
	end
end
```

## Revisar los Documents en Luxe

Para ver los documents de los protocols de un physician en Luxe hay que ir al perfil y buscar la sección "Documents". Ahí se clica "manage" y se podrán ver, si los hay.

Physician sin documentos aunque tiene Protocol asociado:
![[001.documents.png]]

Physician con documentos:
![[002.no.documents.png]]

# Phase Periods

Etiquetas: #luna_help_desk

Q: ¿Qué son estas cosas en `phase_periods`?

```
phase_periods: ["0:0"]

phase_periods: ["-10:2", "3:10", "11:23", "24:37", "38:48"]
```

Según Claude:

> Surgery-based protocols: Use time ranges like ["-10:0", "1:10", "11:23"] (days before/after surgery).
> Visit-based protocols: Use visit counts like ["0:0", "1:1", "2:2"] (visit sequences).

¿Cómo se sabe si es Visit-base o Surgery-based? Por la key `phase_measure`.

Normalmente, cuando es Visit-based, se incluye la key con ese valor en el archivo del protocolo. Ejemplo:
```yaml
phase_periods: ["0:0"]
phase_measure: "completed_visits_count"
```

Cuando no está se asigna la alternativa desde la función correspondiente en el `ProtocolSeeder`:
```ruby
def protocol_data
	protocols_hash.map do |protocol_hash|
		Protocol.new(
			# (...)
			phase_measure: protocol_hash["phase_measure"] || "surgery_date_offset_in_days"
		)
	end
end
```

Entonces cuando son Visit-based los `phase_periods` se manejan en duplas cerradas:
```yaml
phase_periods: ["0:0", "1:1", "2:2", "3:3", "4:4", "5:5", "6:6", "7:7", "8:8", "9:9", "14:14", "19:19" ]
```

Y cuando son Surgery-based se manejan en duplas de rangos que no se solapan:
```yaml
phase_periods: ["-10:2", "3:10", "11:23", "24:37", "38:48"]
```

## ¿Diferencia entre Visit-based y Surgery-based?

En Visit-based el rango es en torno al número de la visita.

En Surgery-base el rango hace referencia a los días pre y post cirugía. Si se encuentra un número negativo significa _días antes de la cirugía_. Cuando son números positivos significa _días después de la cirugía_.

## Detalles Técnicos

La función que trata con esto es esta en `ProtocolSeeder`:
```ruby
def protocol_phase_data
	protocol_id_to_protocol = Protocol.all.index_by(&:id)
	protocols_hash.flat_map do |protocol_hash|
		protocol_hash.fetch("phase_periods").each_with_index.map do |phase_period, phase_index|
			lower, upper = phase_period.split(":")
			lower = nil if lower.blank?
			upper = nil if upper.blank?

			raise "Invalid phase period detected in protocol #{protocol_hash.fetch('id')}" if lower.nil? && upper.nil?

			ProtocolPhase.new(
				protocol: protocol_id_to_protocol[protocol_hash.fetch("id")],
				relative_period: lower..upper,
				number: 1 + phase_index
			)
		end
	end
end
```

Ahí se ve que se toman ambos dígitos al separarlos por el dos puntos. Se usan para crear un `ProtocolPhase`. Así son las relaciones.

```ruby
class Protocol < ApplicationRecord
  has_many :phases, class_name: "ProtocolPhase"
end

class ProtocolPhase < ApplicationRecord
  belongs_to :protocol
end
```

El campo `relative_period` es de tipo `int4range`. Se define así en el schema:
```ruby
t.int4range "relative_period", null: false

t.exclusion_constraint "relative_period WITH &&, protocol_id WITH =", using: :gist, name: "protocol_phases_802620237"
```

## ¿Qué significa la restricción de exclusión en `relative_period`?

Significa que dos registros no pueden solaparse. Determinado por `relative_period WITH &&`.

✅ You can have:
    - protocol_id = 1, relative_period = `[1,5)`
    - protocol_id = 1, relative_period = `[5,10)`

❌ You cannot have:
    - protocol_id = 1, relative_period = `[1,5)`
    - protocol_id = 1, relative_period = `[4,8)`  
        (because those ranges overlap)

La expresión `protocol_id WITH =` hace que la comparación solo se dé sin tienen el mismo `protocol_id`.


# NPI y Physicians assignment

