# Entendiendo Legacy Connection Handling

Quiero entender esta configuración para saber porque causa el error
```
undefined method while_preventing_writes for ActiveRecord::ConnectionAdapters::ConnectionHandler
```

cuál es el valor a darle a esta configuración y tener mejor claridad pensando en que en Edge la configuración `config.load_defaults` está apuntando a la versión 6.0 y eventualmente tendrá que cambiar a 7.1.

**Otros Docs Relacionados**

La configuración `config.active_record.legacy_connection_handling` ya la tengo documentada en otros documentos:

- [[004 - Config Defaults#What is config.active_record.legacy_connection_handling?]]
- [[Pruebas de Rails 7#while_preventing_writes is only available on the connection_handler with legacy_connection_handling ✅]]

## Contexto y Documentación

Cuando se hace el upgrade a Rails 7.1.4, este código:
```ruby
if replica_available?
      ActiveRecord::Base.connected_to(role: :reading) do
        ActiveRecord::Base.connection_handler.while_preventing_writes(true, &block)
      end
    else
```

en la clase `DatabaseSwitching` explota al usar el método `while_preventing_writes`. El error dice que ese método no está definido para la clase `ActiveRecord::ConnectionAdapters::ConnectionHandler`.

Los docs de [Rails 7.0.8.4](https://guides.rubyonrails.org/v7.0.8.4/active_record_multiple_databases.html#migrate-to-the-new-connection-handling) tienen una sección que menciona una "nueva API de _connection handling_" y mencionan esta configuración `legacy_connection_handling`. Dice que esta API se agregó desde Rails 6.1+.

Las guías en [Configuring Rails explican que](https://guides.rubyonrails.org/v7.0.8.4/configuring.html#config-active-record-legacy-connection-handling) esta configuración:

> Allows to enable new connection handling API. For applications using multiple databases, this new API provides support for granular connection swapping.

En Rails 6.1 se pone en *false*. En Rails 7.0 se pone en *true*.

## ¿Qué valor tiene esta configuración en Development y Alpha?

Para revisar el valor de este setting hay que ejecutar esto en una consola de Rails:
```ruby
Rails.application.config.active_record.legacy_connection_handling
```

**En development**

En Rails 7.1.4 devuelve nulo:
```
Loading development environment (Rails 7.1.4)

Frame number: 0/13
[1] pry(main)> Rails.application.config.active_record.legacy_connection_handling
=> nil
```

En Rails 7.0.8.4 devuelve nulo:
```
Loading development environment (Rails 7.0.8.4)

Frame number: 0/10
[1] pry(main)> Rails.application.config.active_record.legacy_connection_handling
=> nil
```

Si intento configurarlo en la rama con Rails 7.1.4 obtengo este error:
```
`legacy_connection_handling=': The `legacy_connection_handling` setter was deprecated in 7.0 and removed in 7.1, but is still defined in your configuration. Please remove this call as it no longer has any effect." (ArgumentError)
```