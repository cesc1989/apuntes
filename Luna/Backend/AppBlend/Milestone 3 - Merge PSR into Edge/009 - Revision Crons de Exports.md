# Comandos para Revisar que los Crons exportaron los datos al Data Lake

## Para Alpha

```bash
s3ls --human-readable luna-alpha-workloads-data-lake/application-data/patient-self-report/patient-forms/latest/data.csv

s3ls --human-readable luna-alpha-workloads-data-lake/application-data/patient-self-report/patient-forms-scores/latest/data.csv

s3ls --human-readable luna-alpha-workloads-data-lake/application-data/patient-self-report/patient-forms-mips/latest/data.csv

s3ls --human-readable luna-alpha-workloads-data-lake/business-operations/patient-forms/latest/data.csv

s3ls --human-readable luna-alpha-workloads-data-lake/business-operations/patient-self-report/intake-form-summary/latest/data.csv

s3ls --human-readable luna-alpha-workloads-data-lake/business-operations/patient-nps-scores/latest/data.csv
```

## Para Omega

```bash
s3ls --human-readable luna-omega-workloads-data-lake/application-data/patient-self-report/patient-forms/latest/data.csv

s3ls --human-readable luna-omega-workloads-data-lake/application-data/patient-self-report/patient-forms-scores/latest/data.csv

s3ls --human-readable luna-omega-workloads-data-lake/application-data/patient-self-report/patient-forms-mips/latest/data.csv

s3ls --human-readable luna-omega-workloads-data-lake/business-operations/patient-forms/latest/data.csv

s3ls --human-readable luna-omega-workloads-data-lake/business-operations/patient-self-report/intake-form-summary/latest/data.csv

s3ls --human-readable luna-omega-workloads-data-lake/business-operations/patient-nps-scores/latest/data.csv
```