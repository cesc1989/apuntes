# Apuntes Generales

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

