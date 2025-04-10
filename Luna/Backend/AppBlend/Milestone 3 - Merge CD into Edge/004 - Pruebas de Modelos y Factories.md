# Pruebas de Modelos y Factories cuando el primary key no tiene secuencia por defecto

Debido a la configuración del Logical Replication hubo que quitar el valor por defecto a la llave primaria de las tablas de Clinical Dashboard que fueron copiadas a Edge. Por esto, las pruebas fallan con el error:
```
 ActiveRecord::NotNullViolation:
       PG::NotNullViolation: ERROR:  null value in column "id" of relation "clinical_dashboards" violates not-null constraint
       DETAIL:  Failing row contains (null, Kratos Rodriguez, f344955c-6d23-40d9-9eef-8e723f3ce41e, physician, kratos@gmail.com,toskra@aol.com, 2018-10-14 11:00:00, 2018-10-14 11:00:00).
```

Como ya se vio en [[004 - Modelos y Factories]] esto se puede solucionar con unos ajustes en el Factory o con un Concern para el modelo.

En realidad la solución solo se da con el Concern en el modelo.

## Factories y Concern para llave primaria

Así quedó la factory `dashboards.rb`:
```ruby
FactoryBot.define do
  factory :dashboard, class: "ClinicalDashboard::Dashboard" do
    provider_name { "Kratos Rodriguez" }
    provider_id { SecureRandom.uuid }
    provider_kind { "physician" }
    provider_emails { "kratos@gmail.com,toskra@aol.com" }

    # (...)
  end
end
```

Y así está el concern:
```ruby
module ClinicalDashboardPrimaryKey
  extend ActiveSupport::Concern

  private

  def generate_unique_bigint_id
    self.id ||= self.class.maximum(:id).to_i + 1
  end

  def generate_uuid
    self.id ||= SecureRandom.uuid
  end
end
```

El método que necesito se usa en un callback `before_create` en el modelo correspondiente y las pruebas pasan.

No es necesario agregar ni `sequence` al ID ni un `initialize_with` como sugiere [[004 - Modelos y Factories#Tablas de ID bigint no tienen secuencia por Logical Replication]]