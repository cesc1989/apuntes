# 003 - Export Therapist Inactive Reason to therapist-forward-fill table

## Pruebas

Para disparar manualmente el worker en una consola de Rails:
```ruby
Athena::TherapistForwardFillWriterWorker.perform_async
```

Query de prueba en Athena:
```sql
SELECT *
FROM "business-operations"."therapist_forward_fill"
where inactive_reason_key is not null
limit 20;
```