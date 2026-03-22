# Notas de Upgrade a Rails 7.2.3

## Error `undefined method 'schema_migration' for an instance of ActiveRecord::ConnectionAdapters::SQLite3Adapter`

Esto cuando corrí las pruebas:
```bash
Failure/Error: DatabaseCleaner.clean_with :truncation

NoMethodError:
  undefined method 'schema_migration' for an instance of ActiveRecord::ConnectionAdapters::SQLite3Adapter
# /Users/francisco/.gem/ruby/3.4.8/gems/database_cleaner-active_record-2.1.0/lib/database_cleaner/active_record/base.rb:19:in 'DatabaseCleaner::ActiveRecord::Base.migration_table_name'
```

### Solución 🟢

Subir gema database_cleaner a la versión 2.1.0 -> https://github.com/DatabaseCleaner/database_cleaner-active_record/issues/83#issuecomment-1464759691