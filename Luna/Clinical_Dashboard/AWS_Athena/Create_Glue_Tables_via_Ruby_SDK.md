# Create Glue Tables via Ruby SDK
Trying to create Glue tables via the CLI had me struck on an error about JSON parameter. Seems simpler with the Ruby SDK.

## Notes
- Using `#update_table` looks cumbersome as it requires many attributes.
    - Feels better to delete and create when editing a table. But need to proceed with caution.
- Had to install gem `gem 'aws-sdk-glue', '1.22.0'` in order to access it from the rails console (for simplicity)


## Links
- [Ruby gem Docs](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/Glue/Client.html)
- `[#create_table](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/Glue/Client.html#create_table-instance_method)` [docs](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/Glue/Client.html#create_table-instance_method)
- `[#get_tables](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/Glue/Client.html#get_tables-instance_method)` [docs](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/Glue/Client.html#get_tables-instance_method)


# For Staging
## NPS Scores table
    client = Aws::Glue::Client.new
    client.create_table(
      database_name: "patient-forms-production-db",
      table_input: {
        name: "nps_scores",
        description: "NPS scores by patients in Progress Forms. Created by Francisco Quintero",
        storage_descriptor: {
          columns: [
            { name: "id", type: "bigint" },
            { name: "patient_id", type: "string" },
            { name: "form_type", type: "string" },
            { name: "injury_name", type: "string" },
            { name: "care_plan_id", type: "string" },
            { name: "nps_0", type: "bigint" },
            { name: "nps_1", type: "bigint" },
            { name: "nps_2", type: "bigint" },
            { name: "nps_3", type: "bigint" },
            { name: "nps_4", type: "bigint" },
            { name: "nps_5", type: "bigint" },
            { name: "nps_6", type: "bigint" },
            { name: "nps_7", type: "bigint" },
            { name: "nps_8", type: "bigint" },
            { name: "nps_9", type: "bigint" },
            { name: "form_type_0", type: "string" },
            { name: "form_type_1", type: "string" },
            { name: "form_type_2", type: "string" },
            { name: "form_type_3", type: "string" },
            { name: "form_type_4", type: "string" },
            { name: "form_type_5", type: "string" },
            { name: "form_type_6", type: "string" },
            { name: "form_type_7", type: "string" },
            { name: "form_type_8", type: "string" },
            { name: "form_type_9", type: "string" },
            { name: "completed_at_0", type: "string" },
            { name: "created_at_0", type: "string" },
            { name: "completed_at_1", type: "string" },
            { name: "created_at_1", type: "string" },
            { name: "completed_at_2", type: "string" },
            { name: "created_at_2", type: "string" },
            { name: "completed_at_3", type: "string" },
            { name: "created_at_3", type: "string" },
            { name: "completed_at_4", type: "string" },
            { name: "created_at_4", type: "string" },
            { name: "completed_at_5", type: "string" },
            { name: "created_at_5", type: "string" },
            { name: "completed_at_6", type: "string" },
            { name: "created_at_6", type: "string" },
            { name: "completed_at_7", type: "string" },
            { name: "created_at_7", type: "string" },
            { name: "completed_at_8", type: "string" },
            { name: "created_at_8", type: "string" },
            { name: "completed_at_9", type: "string" },
            { name: "created_at_9", type: "string" }
          ],
          input_format: "org.apache.hadoop.mapred.TextInputFormat",
          output_format: "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
          location: "s3://luna-staging-data-lake/patient-forms-pipeline/patient-scores/staging/",
          serde_info: {
            serialization_library: "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe",
            parameters: { "field.delim": "," }
          }
        },
        parameters: {
          "classification": "csv",
          "skip.header.line.count": "1"
        }
      }
    )


## Thumbs Up/Down table
    client = Aws::Glue::Client.new
    client.create_table(
      database_name: "luxe-production-db",
      table_input: {
        name: "thumbs_appointments",
        description: "Thumbs up/down ratings. Created by Francisco Quintero",
        storage_descriptor: {
          columns: [
            { name: "care_plan_id", type: "string" },
            { name: "patient_id", type: "string" },
            { name: "therapist_name", type: "string" },
            { name: "appointment_id", type: "string" },
            { name: "created_at", type: "string" },
            { name: "scheduled_at", type: "string" },
            { name: "visit_type", type: "string" },
            { name: "status", type: "string" },
            { name: "patient_rating", type: "string" },
            { name: "physician_id", type: "string" }
          ],
          input_format: "org.apache.hadoop.mapred.TextInputFormat",
          output_format: "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
          location: "s3://luna-staging-data-lake/luxe-pipeline/staging/appointments/latest/",
          serde_info: {
            serialization_library: "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe",
            parameters: { "field.delim": "|" }
          }
        },
        parameters: {
          "classification": "csv",
          "skip.header.line.count": "1"
        }
      }
    )
# For UAT
## NPS Scores table
    
## Thumbs up/down table
    client = Aws::Glue::Client.new
    client.create_table(
      database_name: "luxe-production-db",
      table_input: {
        name: "thumbs_appointments",
        description: "Thumbs up/down ratings. Created by Francisco Quintero",
        storage_descriptor: {
          columns: [
            { name: "care_plan_id", type: "string" },
            { name: "patient_id", type: "string" },
            { name: "therapist_name", type: "string" },
            { name: "appointment_id", type: "string" },
            { name: "created_at", type: "string" },
            { name: "scheduled_at", type: "string" },
            { name: "visit_type", type: "string" },
            { name: "status", type: "string" },
            { name: "patient_rating", type: "string" },
            { name: "physician_id", type: "string" }
          ],
          input_format: "org.apache.hadoop.mapred.TextInputFormat",
          output_format: "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
          location: "s3://luna-uat-data-lake/luxe-pipeline/uat/appointments/latest/",
          serde_info: {
            serialization_library: "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe",
            parameters: { "field.delim": "|" }
          }
        },
        parameters: {
          "classification": "csv",
          "skip.header.line.count": "1"
        }
      }
    )
# For Production
## NPS Scores table
    client = Aws::Glue::Client.new
    client.create_table(
      database_name: "patient-forms-production-db",
      table_input: {
        name: "nps_scores",
        description: "NPS scores by patients in Progress Forms. Created by Francisco Quintero",
        storage_descriptor: {
          columns: [
            { name: "id", type: "bigint" },
            { name: "patient_id", type: "string" },
            { name: "form_type", type: "string" },
            { name: "injury_name", type: "string" },
            { name: "care_plan_id", type: "string" },
            { name: "nps_0", type: "bigint" },
            { name: "nps_1", type: "bigint" },
            { name: "nps_2", type: "bigint" },
            { name: "nps_3", type: "bigint" },
            { name: "nps_4", type: "bigint" },
            { name: "nps_5", type: "bigint" },
            { name: "nps_6", type: "bigint" },
            { name: "nps_7", type: "bigint" },
            { name: "nps_8", type: "bigint" },
            { name: "nps_9", type: "bigint" },
            { name: "form_type_0", type: "string" },
            { name: "form_type_1", type: "string" },
            { name: "form_type_2", type: "string" },
            { name: "form_type_3", type: "string" },
            { name: "form_type_4", type: "string" },
            { name: "form_type_5", type: "string" },
            { name: "form_type_6", type: "string" },
            { name: "form_type_7", type: "string" },
            { name: "form_type_8", type: "string" },
            { name: "form_type_9", type: "string" },
            { name: "completed_at_0", type: "string" },
            { name: "created_at_0", type: "string" },
            { name: "completed_at_1", type: "string" },
            { name: "created_at_1", type: "string" },
            { name: "completed_at_2", type: "string" },
            { name: "created_at_2", type: "string" },
            { name: "completed_at_3", type: "string" },
            { name: "created_at_3", type: "string" },
            { name: "completed_at_4", type: "string" },
            { name: "created_at_4", type: "string" },
            { name: "completed_at_5", type: "string" },
            { name: "created_at_5", type: "string" },
            { name: "completed_at_6", type: "string" },
            { name: "created_at_6", type: "string" },
            { name: "completed_at_7", type: "string" },
            { name: "created_at_7", type: "string" },
            { name: "completed_at_8", type: "string" },
            { name: "created_at_8", type: "string" },
            { name: "completed_at_9", type: "string" },
            { name: "created_at_9", type: "string" }
          ],
          input_format: "org.apache.hadoop.mapred.TextInputFormat",
          output_format: "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
          # location: "s3://luna-staging-data-lake/patient-forms-pipeline/patient-scores/staging/",
          location: "s3://luna-data-lake/patient-forms-pipeline/patient-scores/production/",
          serde_info: {
            serialization_library: "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe",
            parameters: { "field.delim": "," }
          }
        },
        parameters: {
          "classification": "csv",
          "skip.header.line.count": "1"
        }
      }
    )


## Thumbs Up/Down table

Currently being auto generated by the crawler but it’s not assigning proper column names.

Manually create with:

    client = Aws::Glue::Client.new
    client.create_table(
      database_name: "luxe-production-db",
      table_input: {
        name: "thumbs_appointments",
        description: "Thumbs up/down ratings. Created by Francisco Quintero",
        storage_descriptor: {
          columns: [
            { name: "care_plan_id", type: "string" },
            { name: "patient_id", type: "string" },
            { name: "therapist_name", type: "string" },
            { name: "appointment_id", type: "string" },
            { name: "created_at", type: "string" },
            { name: "scheduled_at", type: "string" },
            { name: "visit_type", type: "string" },
            { name: "status", type: "string" },
            { name: "patient_rating", type: "string" },
            { name: "physician_id", type: "string" }
          ],
          input_format: "org.apache.hadoop.mapred.TextInputFormat",
          output_format: "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
          location: "s3://luna-data-lake/luxe-pipeline/production/appointments/latest/",
          serde_info: {
            serialization_library: "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe",
            parameters: { "field.delim": "|" }
          }
        },
        parameters: {
          "classification": "csv",
          "skip.header.line.count": "1"
        }
      }
    )

