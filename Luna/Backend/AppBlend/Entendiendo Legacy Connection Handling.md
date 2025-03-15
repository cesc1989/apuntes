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

# ¿Cómo corregir el error de `while_preventing_writes` no definido?

## Opción 1: Usando `preventing_writes?`

Efectivamente, viendo los docs de la API de Rails me doy cuenta que el método `while_preventing_writes` existe en [Rails 6.1](https://api.rubyonrails.org/v6.1/classes/ActiveRecord/ConnectionAdapters/ConnectionHandler.html#method-i-while_preventing_writes), en [Rails 7.0.8.4](https://api.rubyonrails.org/v7.0.8.4/classes/ActiveRecord/ConnectionAdapters/ConnectionHandler.html#method-i-while_preventing_writes) pero no existe más en [Rails 7.1.4](https://api.rubyonrails.org/v7.1.4/classes/ActiveRecord/ConnectionAdapters/ConnectionHandler.html)

¿Cuál es la alternativa o método(s) a usar?

En las [guías _Multiple Databases_ de la versión 7.0.8.4](https://guides.rubyonrails.org/v7.0.8.4/active_record_multiple_databases.html#migrate-to-the-new-connection-handling) dice que al migrar a esta nueva API de _connection handling_ hay que cambiar algunos métodos si estamos en un setup de varias bases de datos. La clave aquí es que mencionan esto:

> Calls to `ActiveRecord::Base.connection_handler.prevent_writes` will need to be updated to `ActiveRecord::Base.connection.preventing_writes?`

> [!Note]
> El método `prevent_writes` no lo encuentro en la documentación de la API de Rails.

Este fue el cambio que hice:
```diff
def self.on_read_database(&block)
  if replica_available?
    ActiveRecord::Base.connected_to(role: :reading) do
-     ActiveRecord::Base.connection_handler.while_preventing_writes(true, &block)
+     yield if ActiveRecord::Base.connection.preventing_writes?
    end
  else
    yield
  end
end
```

Esto lo pude replicar lanzando el servidor Rails y luego ir a la página Concierge -> At Risk Reports. Cuando se usa `while_preventing_writes` explota el servidor con el error que nos trajo aquí. Una vez cambio al nuevo método todo carga en orden.

> [!Note]
> Q: ¿sí está funcionando o simplemente no explota?
> R: 

La decisión de la forma que lo hice es porque:

- El uso de `on_read_database` siempre se da pasando un *bloque* que contiene llamados a objetos Active Record.
- Cuando no hay replica simplemente se pasa el bloque a `yield`.
- El uso de `while_preventing_writes` se da pasando el mismo bloque de la función principal. En su [implementación](https://api.rubyonrails.org/v7.1.4/classes/ActiveRecord/ConnectionHandling.html#method-i-while_preventing_writes), este método pasa el parámetro bloque a otra función.
- Por su parte, `preventing_writes` retorna true si se está leyendo la base de datos replica. [Ver docs](https://api.rubyonrails.org/v7.1.4/classes/ActiveRecord/ConnectionAdapters/AbstractAdapter.html#method-i-preventing_writes-3F).

## Opción 2: Forma Ideal, usando opción del método `connected_to`

Esto no estaba en la versión 6.1. Desde la [versión 7.0+](https://guides.rubyonrails.org/v7.0.8.4/active_record_multiple_databases.html#using-manual-connection-switching) es que se puede hacer esto:
```ruby
ActiveRecord::Base.connected_to(role: :reading, prevent_writes: true) do
end
```

[Ver Docs](https://guides.rubyonrails.org/v7.0.8.4/active_record_multiple_databases.html#using-manual-connection-switching) de `connected_to`.

Y es la forma correcta de hacerlo ya que es lo que provee el framework.