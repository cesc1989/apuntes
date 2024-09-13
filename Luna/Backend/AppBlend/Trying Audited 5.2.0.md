# Trying Audited 5.2.0

> [!INFO]
> En alguna versión después de la 4.9.0, cambiaron la clase del atributo `audited_changes` de `ActiveSupport::HashWithIndifferentAccess` a `Hash`

Anteriormente accedían los valores del hash como símbolo:
```ruby
def accepting_new_patients_changed_at
    audits.select do |audit|
      audit.audited_changes.include? :accepting_new_patients
    end.last&.created_at
end
```

En la versión 5.2.0, el hash `audited_changes` devuelve las llaves como strings:
```ruby
[#<Audited::Audit:0x00007f49ffba86c8
  auditable_id: "1ee9b8d1-8353-40cb-a9a0-0044ee450f60",
  auditable_type: "Therapist",
  username: "fd879de0-a4d0-4123-9701-862a9aedd15d",
  action: "create",
  audited_changes:
   {"profile_description"=>"",
    "appointments_per_week"=>29,
    "ssn_last_4"=>"3468",
    "scheduling_patient_id"=>nil,
    "national_provider_identifier"=>"1053797893",
    "accepting_new_patients"=>"closed_to_new_patients",
    "year_of_most_recently_sent_tax_forms"=>nil,
    "booking_enabled"=>true,
    "link_emr_reminder"=>false},
  version: 1,
  comment: nil,
  remote_address: "207.7.135.68",
  request_uuid: "2dac066b-c74f-4d73-acd6-c957af1d8e53",
  created_at: Sat, 30 Mar 2024 06:54:58.544301000 PDT -07:00>]
```

> Quité elementos para simplificar

# ¿Qué pasa con Audited y Therapist?

## Contexto

El modelo therapist define el enum: `accepting_new_patients`
```ruby
enum accepting_new_patients: {
    closed_to_new_patients: 0,
    open_to_new_patients: 1,
    open_to_waitlist_patients: 2
}
```

que tiene por defecto el valor 0:
```ruby
t.integer "accepting_new_patients", default: 0, null: false
```

En la prueba:
```ruby
describe "#accepting_new_patients_changed_at" do
    it "returns the most recent time #accepting_new_patients was changed" do
      Timecop.freeze do
        therapist = create(:therapist)
        therapist.save!
        # no changes yet to report
        expect(therapist.accepting_new_patients_changed_at).to be_nil

        therapist.update!(
          # 'change' the value to itself
          accepting_new_patients: therapist.accepting_new_patients,
          # change an unrelated attribute
          credentials: "ayy lmao"
        )
        # still no recorded change
        expect(therapist.accepting_new_patients_changed_at).to be_nil

        therapist.update!(
          accepting_new_patients: :closed_to_new_patients
        )
        # changed just now
        expect(therapist.accepting_new_patients_changed_at).to eq(Time.current)
      end
    end
end
```

En la versión anterior se espera que esto se cumpla:
```ruby
expect(therapist.accepting_new_patients_changed_at).to be_nil
expect(therapist.accepting_new_patients_changed_at).to be_nil
```

La factory se configura así:
```ruby
accepting_new_patients { :open_to_new_patients }
```

Y este es el método `accepting_new_patients_changed_at`:
```ruby
def accepting_new_patients_changed_at
    audits.select do |audit|
      audit.audited_changes.include? :accepting_new_patients
    end.last&.created_at
end
```

Se espera que los expectations sean nulos porque al crearse no hay ningún audit que incluya la llave `accepting_new_patients`.

Sin embargo, eso cambia en la versión 5.2.0

## Versión 4.9.0

Este es el viaje de los audits que se ven en esa prueba:
```ruby
[
    [0] #<Audited::Audit:0x0000000148f762e8> {
                     :id => 195,
           :auditable_id => "ef5f4ed0-1a6b-45aa-b4ab-d79415fcee5a",
         :auditable_type => "Therapist",
                 :action => "create",
        :audited_changes => {
                          "accepting_new_patients" => "open_to_new_patients",
        },
                :version => 1,
                :comment => nil,
         :remote_address => nil,
           :request_uuid => "45750444-7f5d-4f44-b207-919d03cbcf89",
             :created_at => Sun, 14 Oct 2018 11:00:00.000000000 UTC +00:00
    },
    [1] #<Audited::Audit:0x000000014a23dca0> {
                     :id => 196,
           :auditable_id => "ef5f4ed0-1a6b-45aa-b4ab-d79415fcee5a",
         :auditable_type => "Therapist",
                 :action => "update",
        :audited_changes => {
            "credentials" => [
                [0] nil,
                [1] "ayy lmao"
            ]
        },
                :version => 2,
                :comment => nil,
         :remote_address => nil,
           :request_uuid => "aa4e6a22-355e-4169-86c7-8e2dca220939",
             :created_at => Sun, 14 Oct 2018 11:00:00.000000000 UTC +00:00
    },
    [2] #<Audited::Audit:0x000000014a67e918> {
                     :id => 197,
           :auditable_id => "ef5f4ed0-1a6b-45aa-b4ab-d79415fcee5a",
         :auditable_type => "Therapist",
                 :action => "update",
        :audited_changes => {
            "accepting_new_patients" => [
                [0] "open_to_new_patients",
                [1] "closed_to_new_patients"
            ]
        },
                :version => 3,
                :comment => nil,
         :remote_address => nil,
           :request_uuid => "209c07ea-24d3-43e1-9efb-a3137d674204",
             :created_at => Sun, 14 Oct 2018 11:00:00.000000000 UTC +00:00
    }
]
```

En el elemento 0, es cuando se crea el therapist. Vemos como:
- en `audited_changes` está la llave como string. No como símbolo.
	- Por eso no lo encuentra en la 1era vez.

En el elemento 1, en la prueba se actualiza `accepting_new_patients` con si mismo así que no cambia y por eso, ¿no se refleja en `audited_changes`?

Finalmente, en el elemento 2, se cambia el valor del enum por lo tanto si está `accepting_new_patients` en `audited_changes` pero sigue apareciendo como string. ¿Qué hace que lo pueda acceder como símbolo?

Aquí está la clave:
```ruby
audit.audited_changes.class
ActiveSupport::HashWithIndifferentAccess
```

Veamos en la 5.2.0. En esta versión:
```ruby
audit.audited_changes.class.name

"Hash"
"Hash"
"Hash"
"Hash"
"Hash"
"Hash"
```

Lo que no encuentro es en que momento hicieron el cambio.