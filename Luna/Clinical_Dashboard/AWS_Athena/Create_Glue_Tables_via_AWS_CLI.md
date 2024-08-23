# Create Glue Tables via AWS CLI

# Tables

Links:

- [create-table](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/glue/create-table.html)
- [get-table](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/glue/get-table.html)


# Get JSON for an existing table
    aws glue get-table --database-name luxe-production-db --name physicians --profile alpha >> ~/Downloads/physicians.json


> **Notice** `**--profile alpha**`  **in the example commands.**


# Edge Tables
## appointments table
    aws glue create-table \
        --cli-input-json '{
        "DatabaseName": "application-data",
        "TableInput": {
            "Name": "appointments",
            "Description": "Thumbs up/down ratings. Created by Francisco Quintero",
            "StorageDescriptor": {
                "Columns": [
                    { "Name": "care_plan_id", "Type": "string" },
                    { "Name": "patient_id", "Type": "string" },
                    { "Name": "therapist_name", "Type": "string" },
                    { "Name": "appointment_id", "Type": "string" },
                    { "Name": "created_at", "Type": "string" },
                    { "Name": "scheduled_at", "Type": "string" },
                    { "Name": "visit_type", "Type": "string" },
                    { "Name": "status", "Type": "string" },
                    { "Name": "patient_rating", "Type": "string" },
                    { "Name": "physician_id", "Type": "string" }
                ],
                "Location": "s3://luna-alpha-workloads-data-lake/application-data/edge/appointments/",
                "InputFormat": "org.apache.hadoop.mapred.TextInputFormat",
                "OutputFormat": "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
                "SerdeInfo": {
                    "SerializationLibrary": "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe",
                    "Parameters": {
                        "field.delim": "|"
                    }
                }
            },
            "Parameters": {
                "classification": "csv",
                "skip.header.line.count": "1"
            }
        }
    }' \
      --profile alpha


## clinics
    aws glue create-table \
        --cli-input-json '{
        "DatabaseName": "application-data",
        "TableInput": {
            "Name": "clinics",
            "Description": "Created by Francisco Quintero",
            "StorageDescriptor": {
                "Columns": [
                    { "Name": "id", "Type": "string" },
                    { "Name": "key", "Type": "string" },
                    { "Name": "practice_id", "Type": "string" },
                    { "Name": "tax_identification_number", "Type": "string" },
                    { "Name": "national_provider_identifier", "Type": "string" },
                    { "Name": "provider_name", "Type": "string" }
                ],
                "Location": "s3://luna-alpha-workloads-data-lake/application-data/edge/clinics/",
                "InputFormat": "org.apache.hadoop.mapred.TextInputFormat",
                "OutputFormat": "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
                "SerdeInfo": {
                    "SerializationLibrary": "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe",
                    "Parameters": {
                        "field.delim": "|"
                    }
                }
            },
            "Parameters": {
                "classification": "csv",
                "skip.header.line.count": "1"
            }
        }
    }' \
      --profile alpha


## patients
    aws glue create-table \
        --cli-input-json '{
        "DatabaseName": "application-data",
        "TableInput": {
            "Name": "patients",
            "Description": "Created by Francisco Quintero",
            "StorageDescriptor": {
                "Columns": [
                    { "Name": "id", "Type": "string" },
                    { "Name": "first_name", "Type": "string" },
                    { "Name": "last_name", "Type": "string" },
                    { "Name": "gender", "Type": "string" },
                    { "Name": "age", "Type": "double" },
                    { "Name": "practice_id", "Type": "string" },
                    { "Name": "discharged", "Type": "boolean" },
                    { "Name": "physician_id", "Type": "string" },
                    { "Name": "completed_visits_count", "Type": "bigint" },
                    { "Name": "pending_visits_count", "Type": "bigint" },
                    { "Name": "earliest_visit_date", "Type": "string" },
                    { "Name": "latest_visit_date", "Type": "string" },
                    { "Name": "latest_rom_form", "Type": "string" },
                    { "Name": "date_of_birth", "Type": "string" },
                    { "Name": "surgery_date", "Type": "string" },
                    { "Name": "latest_rom_form_flexion", "Type": "bigint" },
                    { "Name": "latest_rom_form_extension", "Type": "bigint" },
                    { "Name": "cancelled_appointments_count", "Type": "bigint" },
                    { "Name": "no_showed_appointments_count", "Type": "bigint" },
                    { "Name": "clinic_id", "Type": "string" }
                ],
                "Location": "s3://luna-alpha-workloads-data-lake/application-data/edge/patients/",
                "InputFormat": "org.apache.hadoop.mapred.TextInputFormat",
                "OutputFormat": "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
                "SerdeInfo": {
                    "SerializationLibrary": "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe",
                    "Parameters": {
                        "field.delim": "|"
                    }
                }
            },
            "Parameters": {
                "classification": "csv",
                "skip.header.line.count": "1"
            }
        }
    }' \
      --profile alpha


## physician_groups
    aws glue create-table \
        --cli-input-json '{
        "DatabaseName": "application-data",
        "TableInput": {
            "Name": "physician_groups",
            "Description": "Created by Francisco Quintero",
            "StorageDescriptor": {
                "Columns": [
                    { "Name": "id", "Type": "string" },
                    { "Name": "name", "Type": "string" },
                    { "Name": "can_sign_patient_pocs", "Type": "boolean" },
                    { "Name": "can_view_patient_charts", "Type": "boolean" },
                    { "Name": "can_view_assigned_physicians", "Type": "boolean" }
                ],
                "Location": "s3://luna-alpha-workloads-data-lake/application-data/edge/physician_groups/",
                "InputFormat": "org.apache.hadoop.mapred.TextInputFormat",
                "OutputFormat": "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
                "SerdeInfo": {
                    "SerializationLibrary": "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe",
                    "Parameters": {
                        "field.delim": "|"
                    }
                }
            },
            "Parameters": {
                "classification": "csv",
                "skip.header.line.count": "1"
            }
        }
    }' \
      --profile alpha


## physicians
    aws glue create-table \
        --cli-input-json '{
        "DatabaseName": "application-data",
        "TableInput": {
            "Name": "physicians",
            "Description": "Created by Francisco Quintero",
            "StorageDescriptor": {
                "Columns": [
                    { "Name": "id", "Type": "string" },
                    { "Name": "prefix", "Type": "string" },
                    { "Name": "first_name", "Type": "string" },
                    { "Name": "last_name", "Type": "string" },
                    { "Name": "email", "Type": "string" },
                    { "Name": "physician_group_id", "Type": "string" },
                    { "Name": "can_sign_patient_pocs", "Type": "boolean" },
                    { "Name": "can_view_patient_charts", "Type": "boolean" },
                    { "Name": "can_view_assigned_physicians", "Type": "boolean" }
                ],
                "Location": "s3://luna-alpha-workloads-data-lake/application-data/edge/physicians/",
                "InputFormat": "org.apache.hadoop.mapred.TextInputFormat",
                "OutputFormat": "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
                "SerdeInfo": {
                    "SerializationLibrary": "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe",
                    "Parameters": {
                        "field.delim": "|"
                    }
                }
            },
            "Parameters": {
                "classification": "csv",
                "skip.header.line.count": "1"
            }
        }
    }' \
      --profile alpha


## practices
    aws glue create-table \
        --cli-input-json '{
        "DatabaseName": "application-data",
        "TableInput": {
            "Name": "practices",
            "Description": "Created by Francisco Quintero",
            "StorageDescriptor": {
                "Columns": [
                    { "Name": "id", "Type": "string" },
                    { "Name": "name", "Type": "string" },
                    { "Name": "code", "Type": "string" },
                    { "Name": "parent_id", "Type": "string" },
                    { "Name": "can_sign_patient_pocs", "Type": "boolean" },
                    { "Name": "can_view_patient_charts", "Type": "boolean" },
                    { "Name": "can_view_assigned_physicians", "Type": "boolean" }
                ],
                "Location": "s3://luna-alpha-workloads-data-lake/application-data/edge/practices/",
                "InputFormat": "org.apache.hadoop.mapred.TextInputFormat",
                "OutputFormat": "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
                "SerdeInfo": {
                    "SerializationLibrary": "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe",
                    "Parameters": {
                        "field.delim": "|"
                    }
                }
            },
            "Parameters": {
                "classification": "csv",
                "skip.header.line.count": "1"
            }
        }
    }' \
      --profile alpha


# Patient Self Report Tables
## patient-forms
    aws glue create-table \
        --cli-input-json '{
        "DatabaseName": "application-data",
        "TableInput": {
            "Name": "patient-forms",
            "Description": "Created by Francisco Quintero",
            "StorageDescriptor": {
                "Columns": [
                    { "Name": "id", "Type": "bigint" },
                    { "Name": "internal_id", "Type": "string" },
                    { "Name": "quality_of_life", "Type": "string" },
                    { "Name": "injury_name", "Type": "string" },
                    { "Name": "forms_count", "Type": "bigint" },
                    { "Name": "completed_forms_count", "Type": "bigint" },
                    { "Name": "form_type", "Type": "string" },
                    { "Name": "pain_0", "Type": "bigint" },
                    { "Name": "psfs_0", "Type": "bigint" },
                    { "Name": "answers_0", "Type": "bigint" },
                    { "Name": "completed_at_0", "Type": "string" },
                    { "Name": "created_at_0", "Type": "string" },
                    { "Name": "pain_1", "Type": "bigint" },
                    { "Name": "psfs_1", "Type": "bigint" },
                    { "Name": "answers_1", "Type": "bigint" },
                    { "Name": "completed_at_1", "Type": "string" },
                    { "Name": "created_at_1", "Type": "string" },
                    { "Name": "pain_2", "Type": "bigint" },
                    { "Name": "psfs_2", "Type": "bigint" },
                    { "Name": "answers_2", "Type": "bigint" },
                    { "Name": "completed_at_2", "Type": "string" },
                    { "Name": "created_at_2", "Type": "string" },
                    { "Name": "pain_3", "Type": "bigint" },
                    { "Name": "psfs_3", "Type": "bigint" },
                    { "Name": "answers_3", "Type": "bigint" },
                    { "Name": "completed_at_3", "Type": "string" },
                    { "Name": "created_at_3", "Type": "string" },
                    { "Name": "pain_4", "Type": "bigint" },
                    { "Name": "psfs_4", "Type": "bigint" },
                    { "Name": "answers_4", "Type": "bigint" },
                    { "Name": "completed_at_4", "Type": "string" },
                    { "Name": "created_at_4", "Type": "string" },
                    { "Name": "pain_5", "Type": "bigint" },
                    { "Name": "psfs_5", "Type": "bigint" },
                    { "Name": "answers_5", "Type": "bigint" },
                    { "Name": "completed_at_5", "Type": "string" },
                    { "Name": "created_at_5", "Type": "string" },
                    { "Name": "pain_6", "Type": "bigint" },
                    { "Name": "psfs_6", "Type": "bigint" },
                    { "Name": "answers_6", "Type": "bigint" },
                    { "Name": "completed_at_6", "Type": "string" },
                    { "Name": "created_at_6", "Type": "string" },
                    { "Name": "pain_7", "Type": "bigint" },
                    { "Name": "psfs_7", "Type": "bigint" },
                    { "Name": "answers_7", "Type": "bigint" },
                    { "Name": "completed_at_7", "Type": "string" },
                    { "Name": "created_at_7", "Type": "string" },
                    { "Name": "pain_8", "Type": "bigint" },
                    { "Name": "psfs_8", "Type": "bigint" },
                    { "Name": "answers_8", "Type": "bigint" },
                    { "Name": "completed_at_8", "Type": "string" },
                    { "Name": "created_at_8", "Type": "string" },
                    { "Name": "pain_9", "Type": "bigint" },
                    { "Name": "psfs_9", "Type": "bigint" },
                    { "Name": "answers_9", "Type": "bigint" },
                    { "Name": "completed_at_9", "Type": "string" },
                    { "Name": "created_at_9", "Type": "string" },
                    { "Name": "care_plan_id", "Type": "string" },
                    { "Name": "progress_modality_0", "Type": "string" },
                    { "Name": "progress_modality_1", "Type": "string" },
                    { "Name": "progress_modality_2", "Type": "string" },
                    { "Name": "progress_modality_3", "Type": "string" },
                    { "Name": "progress_modality_4", "Type": "string" },
                    { "Name": "progress_modality_5", "Type": "string" },
                    { "Name": "progress_modality_6", "Type": "string" },
                    { "Name": "progress_modality_7", "Type": "string" },
                    { "Name": "progress_modality_8", "Type": "string" },
                    { "Name": "progress_modality_9", "Type": "string" },
                    { "Name": "most_recent_form_type", "Type": "string" },
                    { "Name": "symptoms_started_on", "Type": "string" },
                    { "Name": "form_type_0", "Type": "string" },
                    { "Name": "form_type_1", "Type": "string" },
                    { "Name": "form_type_2", "Type": "string" },
                    { "Name": "form_type_3", "Type": "string" },
                    { "Name": "form_type_4", "Type": "string" },
                    { "Name": "form_type_5", "Type": "string" },
                    { "Name": "form_type_6", "Type": "string" },
                    { "Name": "form_type_7", "Type": "string" },
                    { "Name": "form_type_8", "Type": "string" },
                    { "Name": "form_type_9", "Type": "string" }
                ],
                "Location": "s3://luna-alpha-workloads-data-lake/application-data/patient-self-report/patient-forms/",
                "InputFormat": "org.apache.hadoop.mapred.TextInputFormat",
                "OutputFormat": "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
                "SerdeInfo": {
                    "SerializationLibrary": "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe",
                    "Parameters": {
                        "field.delim": ","
                    }
                }
            },
            "Parameters": {
                "classification": "csv",
                "skip.header.line.count": "1"
            }
        }
    }' \
      --profile alpha


## nps_scores
    aws glue create-table \
        --cli-input-json '{
        "DatabaseName": "application-data",
        "TableInput": {
            "Name": "nps_scores",
            "Description": "Created by Francisco Quintero",
            "StorageDescriptor": {
                "Columns": [
                    { "Name": "id", "Type": "bigint" },
                    { "Name": "patient_id", "Type": "string" },
                    { "Name": "form_type", "Type": "string" },
                    { "Name": "injury_name", "Type": "string" },
                    { "Name": "care_plan_id", "Type": "string" },
                    { "Name": "nps_0", "Type": "bigint" },
                    { "Name": "nps_1", "Type": "bigint" },
                    { "Name": "nps_2", "Type": "bigint" },
                    { "Name": "nps_3", "Type": "bigint" },
                    { "Name": "nps_4", "Type": "bigint" },
                    { "Name": "nps_5", "Type": "bigint" },
                    { "Name": "nps_6", "Type": "bigint" },
                    { "Name": "nps_7", "Type": "bigint" },
                    { "Name": "nps_8", "Type": "bigint" },
                    { "Name": "nps_9", "Type": "bigint" },
                    { "Name": "form_type_0", "Type": "string" },
                    { "Name": "form_type_1", "Type": "string" },
                    { "Name": "form_type_2", "Type": "string" },
                    { "Name": "form_type_3", "Type": "string" },
                    { "Name": "form_type_4", "Type": "string" },
                    { "Name": "form_type_5", "Type": "string" },
                    { "Name": "form_type_6", "Type": "string" },
                    { "Name": "form_type_7", "Type": "string" },
                    { "Name": "form_type_8", "Type": "string" },
                    { "Name": "form_type_9", "Type": "string" },
                    { "Name": "completed_at_0", "Type": "string" },
                    { "Name": "created_at_0", "Type": "string" },
                    { "Name": "completed_at_1", "Type": "string" },
                    { "Name": "created_at_1", "Type": "string" },
                    { "Name": "completed_at_2", "Type": "string" },
                    { "Name": "created_at_2", "Type": "string" },
                    { "Name": "completed_at_3", "Type": "string" },
                    { "Name": "created_at_3", "Type": "string" },
                    { "Name": "completed_at_4", "Type": "string" },
                    { "Name": "created_at_4", "Type": "string" },
                    { "Name": "completed_at_5", "Type": "string" },
                    { "Name": "created_at_5", "Type": "string" },
                    { "Name": "completed_at_6", "Type": "string" },
                    { "Name": "created_at_6", "Type": "string" },
                    { "Name": "completed_at_7", "Type": "string" },
                    { "Name": "created_at_7", "Type": "string" },
                    { "Name": "completed_at_8", "Type": "string" },
                    { "Name": "created_at_8", "Type": "string" },
                    { "Name": "completed_at_9", "Type": "string" },
                    { "Name": "created_at_9", "Type": "string" }
                ],
                "Location": "s3://luna-alpha-workloads-data-lake/application-data/patient-self-report/patient-forms-scores/",
                "InputFormat": "org.apache.hadoop.mapred.TextInputFormat",
                "OutputFormat": "org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat",
                "SerdeInfo": {
                    "SerializationLibrary": "org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe",
                    "Parameters": {
                        "field.delim": ","
                    }
                }
            },
            "Parameters": {
                "classification": "csv",
                "skip.header.line.count": "1"
            }
        }
    }' \
      --profile alpha

