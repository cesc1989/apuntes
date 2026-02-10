# Instalación y configuración de Litestack

Se instala de esta forma:

```ruby
gem 'litestack', '~> 0.4.3'
```

Y se ejecuta `bundle install`.

La documentación dice que hay que ejecutar este comando para que se complete la instalación

```
bundle exec rails generate litestack:install
```

pero eso modifica varios archivos:

- cable.yml
- production.rb
- database.yml

Entonces hay que tener en cuenta lo ya hay configurado en esos archivos.

## Litesearch para un modelo

Modelo Comment

```ruby
class Comment < ApplicationRecord
  include Litesearch::Model

  litesearch do |schema|
    schema.fields [:plain_content]
    schema.tokenizer :porter
  end
end
```

Y se busca así en consola de rails

```ruby
Comment.search('termino de búsqueda')
```



# Errores al correr las pruebas

## SQLite3::SQLException: table comments_search_idx_row may not be modified

Corrí las pruebas por 1er vez y salió esto.

```
An error occurred in a `before(:suite)` hook.

Failure/Error: DatabaseCleaner.clean_with :truncation

ActiveRecord::StatementInvalid: 

  SQLite3::SQLException: table comments_search_idx_row may not be modified

# /Users/francisco/.gem/ruby/3.1.0/gems/litestack-0.4.3/lib/litestack/litedb.rb:131:in `initialize'
# /Users/francisco/.gem/ruby/3.1.0/gems/litestack-0.4.3/lib/litestack/litedb.rb:131:in `initialize'
# /Users/francisco/.gem/ruby/3.1.0/gems/litestack-0.4.3/lib/litestack/litedb.rb:68:in `new'
# /Users/francisco/.gem/ruby/3.1.0/gems/litestack-0.4.3/lib/litestack/litedb.rb:68:in `prepare'
# /Users/francisco/.gem/ruby/3.1.0/gems/litestack-0.4.3/lib/litestack/litedb.rb:88:in `execute'
# /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-active_record-2.1.0/lib/database_cleaner/active_record/truncation.rb:96:in `rescue in truncate_table'
# /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-active_record-2.1.0/lib/database_cleaner/active_record/truncation.rb:93:in `truncate_table'
# /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-active_record-2.1.0/lib/database_cleaner/active_record/truncation.rb:100:in `block in truncate_tables'
# /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-active_record-2.1.0/lib/database_cleaner/active_record/truncation.rb:100:in `each'
# /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-active_record-2.1.0/lib/database_cleaner/active_record/truncation.rb:100:in `truncate_tables'
# /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-active_record-2.1.0/lib/database_cleaner/active_record/truncation.rb:25:in `block in clean'
# /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-active_record-2.1.0/lib/database_cleaner/active_record/truncation.rb:21:in `clean'
# /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-core-2.0.1/lib/database_cleaner/cleaner.rb:65:in `clean_with'
# /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-core-2.0.1/lib/database_cleaner/cleaners.rb:40:in `block in clean_with'
# /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-core-2.0.1/lib/database_cleaner/cleaners.rb:40:in `each'
# /Users/francisco/.gem/ruby/3.1.0/gems/database_cleaner-core-2.0.1/lib/database_cleaner/cleaners.rb:40:in `clean_with'

# ./spec/support/database_cleaner.rb:4:in `block (2 levels) in <top (required)>'
# ------------------
# --- Caused by: ---
# SQLite3::SQLException:
#   near "TRUNCATE": syntax error
#   /Users/francisco/.gem/ruby/3.1.0/gems/litestack-0.4.3/lib/litestack/litedb.rb:131:in `initialize'
```

Se corrige quitando la configuración de `truncate` de DatabaseCleaner.

## index configuration not found, either corrupted or not a litesearch index!

Luego de quitar la configuración de DatabaseCleaner me tope con este otro error:

```
Failure/Error:

  litesearch do |schema|
    schema.fields [:plain_content]
    schema.tokenizer :porter
  end

RuntimeError:

  index configuration not found, either corrupted or not a litesearch index!

# /Users/francisco/.gem/ruby/3.1.0/gems/litestack-0.4.3/lib/litestack/litesearch/index.rb:61:in `load_index'
# /Users/francisco/.gem/ruby/3.1.0/gems/litestack-0.4.3/lib/litestack/litesearch/index.rb:18:in `initialize'
# /Users/francisco/.gem/ruby/3.1.0/gems/litestack-0.4.3/lib/litestack/litesearch.rb:25:in `new'
# /Users/francisco/.gem/ruby/3.1.0/gems/litestack-0.4.3/lib/litestack/litesearch.rb:25:in `search_index'
# /Users/francisco/.gem/ruby/3.1.0/gems/litestack-0.4.3/lib/litestack/litesearch/model.rb:44:in `litesearch'
# ./app/models/comment.rb:25:in `<class:Comment>'
# ./app/models/comment.rb:1:in `<top (required)>'
# /Users/francisco/.gem/ruby/3.1.0/gems/zeitwerk-2.6.12/lib/zeitwerk/kernel.rb:30:in `require'
# /Users/francisco/.gem/ruby/3.1.0/gems/zeitwerk-2.6.12/lib/zeitwerk/kernel.rb:30:in `require'
# ./spec/models/comment_spec.rb:5:in `<top (required)>'
```

## Fallos en el CI

Así falló en el github actions

```
SQLite3::IOException: disk I/O error

Migrations are pending. To resolve this issue, run:

bin/rails db:migrate RAILS_ENV=test

You have 11 pending migrations:

20231227145657_devise_create_users.rb
20231227145703_create_active_storage_tables.active_storage.rb
20231227155343_create_channels.rb
20231227155621_create_themes.rb
20231227155855_create_comments.rb
20240107195714_add_twist_id_to_channel.rb
20240107195805_add_twist_id_to_theme.rb
20240111155230_add_twist_id_to_comment.rb
20240118212436_add_service_name_to_active_storage_blobs.active_storage.rb
20240118212437_create_active_storage_variant_records.active_storage.rb
20240118212438_remove_not_null_on_active_storage_blobs_checksum.active_storage.rb
```
