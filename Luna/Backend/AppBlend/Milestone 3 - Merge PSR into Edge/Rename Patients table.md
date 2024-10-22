# Rename Patients table to psr_patients

I'm putting this doc to leave record of how awesome Rails is. I was preparing myself to write several migrations to update indexes and foreign keys after renaming the table but turns out Rails understands it all and did all those changes for me.

Evidence:
```bash
$ rdbm
Calling `DidYouMean::SPELL_CHECKERS.merge!(error_name => spell_checker)' has been deprecated. Please call `DidYouMean.correct_error(erro
r_name, spell_checker)' instead.
Calling `DidYouMean::SPELL_CHECKERS.merge!(error_name => spell_checker)' has been deprecated. Please call `DidYouMean.correct_error(erro
r_name, spell_checker)' instead.
   (0.4ms)  SELECT pg_try_advisory_lock(5827911813384544695)
  ActiveRecord::SchemaMigration Pluck (2.9ms)  SELECT "schema_migrations"."version" FROM "schema_migrations" ORDER BY "schema_migrations
"."version" ASC
Migrating to RenamePatientsTableToPsrPatient (20241022024315)
== 20241022024315 RenamePatientsTableToPsrPatient: migrating ==================
-- rename_table(:patients, :psr_patients)
  TRANSACTION (0.1ms)  BEGIN
   (3.0ms)  ALTER TABLE "patients" RENAME TO "psr_patients"
   (1.1ms)  ALTER INDEX "patients_pkey" RENAME TO "psr_patients_pkey"
   (0.1ms)  ALTER TABLE "public"."patients_id_seq" RENAME TO "psr_patients_id_seq"
   (0.2ms)  ALTER INDEX "index_patients_on_email" RENAME TO "index_psr_patients_on_email"
   (0.1ms)  ALTER INDEX "index_patients_on_internal_id" RENAME TO "index_psr_patients_on_internal_id"
   -> 0.0201s
== 20241022024315 RenamePatientsTableToPsrPatient: migrated (0.0202s) =========

  ActiveRecord::SchemaMigration Create (0.6ms)  INSERT INTO "schema_migrations" ("version") VALUES ($1) RETURNING "version"  [["version"
, "20241022024315"]]
  TRANSACTION (0.5ms)  COMMIT
  ActiveRecord::InternalMetadata Load (1.4ms)  SELECT "ar_internal_metadata".* FROM "ar_internal_metadata" WHERE "ar_internal_metadata".
"key" = $1 LIMIT $2  [["key", "environment"], ["LIMIT", 1]]
   (0.2ms)  SELECT pg_advisory_unlock(5827911813384544695)
  ActiveRecord::SchemaMigration Pluck (0.8ms)  SELECT "schema_migrations"."version" FROM "schema_migrations" ORDER BY "schema_migrations
"."version" ASC
  ActiveRecord::SchemaMigration Pluck (0.1ms)  SELECT "schema_migrations"."version" FROM "schema_migrations" ORDER BY "schema_migrations
"."version" ASC
Annotated (4): app/models/form.rb, app/models/intake_form.rb, app/models/patient.rb, app/models/patient_form_detail.rb
```

Some links for migrations:

- [rename_table](https://api.rubyonrails.org/v7.2.1.1/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-rename_table)
- [rename_index](https://api.rubyonrails.org/v7.2.1.1/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-rename_index)
- [add_foreign_key](https://api.rubyonrails.org/v7.2.1.1/classes/ActiveRecord/ConnectionAdapters/SchemaStatements.html#method-i-add_foreign_key)