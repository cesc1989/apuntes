# Secuencia en Llave Primaria

Las secuencias fueron quitadas para poder completar la configuración para Logical Replication (FOR-344). Para la salida a Producción de la mezcla de Patient Self Report a Edge hay que volver a poner la secuencia en las llaves primarias de las tablas:

1. psr_patients
2. form_types
3. forms
4. intake_forms
5. pain_spots
6. diseases
7. questions
8. option_choices
9. answers
10. aggravating_activities
11. surgeries
12. medications
13. intake_form_diseases
14. intake_form_pain_spots
15. patient_form_details

# Migración

Así me dice que haga ChatGPT:
```ruby
class AddSequenceToPrimaryKey < ActiveRecord::Migration[7.0]
  def up
    table_name = "your_table_name"
    primary_key_column = "id"

    # Create a new sequence
    sequence_name = "#{table_name}_#{primary_key_column}_seq"
    execute <<-SQL
      CREATE SEQUENCE IF NOT EXISTS #{sequence_name} OWNED BY #{table_name}.#{primary_key_column};
    SQL

    # Attach the sequence as the default value for the primary key column
    execute <<-SQL
      ALTER TABLE #{table_name}
      ALTER COLUMN #{primary_key_column}
      SET DEFAULT nextval('#{sequence_name}');
    SQL

    # Set the sequence's value to the maximum current value in the table
    max_id = select_value("SELECT MAX(#{primary_key_column}) FROM #{table_name}")
    max_id = max_id ? max_id.to_i : 0
    execute <<-SQL
      SELECT setval('#{sequence_name}', #{max_id}, true);
    SQL
  end

  def down
    table_name = "your_table_name"
    primary_key_column = "id"

    # Remove the sequence
    sequence_name = "#{table_name}_#{primary_key_column}_seq"
    execute <<-SQL
      ALTER TABLE #{table_name}
      ALTER COLUMN #{primary_key_column} DROP DEFAULT;
    SQL

    execute <<-SQL
      DROP SEQUENCE IF EXISTS #{sequence_name};
    SQL
  end
end
```

Dada la cantidad de tablas lo mejor será hacer un array e iterarlas para correr la consulta SQL.

Nota el uso de
```sql
CREATE SEQUENCE IF NOT EXISTS
```

porque en la copia de alpha la secuencia `psr_patients_id_seq` ya existía pero algunas no.

## Probando en Local

Las pruebas corrieron bien. También pude crear un Form/Patient. Sin embargo, no había quitado el código del concern que asigna un ID aleatorio. En este caso el ID no se generó usando el valor máximo de la secuencia:
```ruby
=> #<PatientSelfReport::Patient:0x000000012eec7e88
 id: 7319873199729848546,

#########

PatientSelfReport::Patient.select("id").max
  PatientSelfReport::Patient Load (40.3ms)  SELECT "psr_patients"."id" FROM "psr_patients"
=> #<PatientSelfReport::Patient:0x000000013d71c968 id: 9222664801011049464>
```

Una vez quito ese código, se empieza a usar el valor de la secuencia para generar el siguiente ID:
```sql
luna_api_development_7=# select last_value from psr_patients_id_seq;
     last_value
---------------------
 9222664801011049464
(1 row)

luna_api_development_7=# select last_value from psr_patients_id_seq;
     last_value
---------------------
 9222664801011049465
(1 row)
```

> [!Note]
> Sobre cómo ver el último valor en una secuencia -> https://stackoverflow.com/a/14886371/1407371

También lo pude comprobar en los registros de Answers para ese form recién creado
```json
{
  "id": 9223260118544186019,
},
{
  "id": 9223260118544186020,
},
{
  "id": 9223260118544186021,
},
{
  "id": 9223260118544186022,
}
```

Vemos como los últimos dígitos van ascendiendo.