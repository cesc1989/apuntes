# Apuntes Apex - Ciclo 004

## Configuración de bundler pre bundle install

Para solucionar el lío con las banderas hay que usar el comando `bundle config set` antes de correr `bundle install`. En este espacio dejo explicación de cada opción que tengo en uso.

Docs: https://bundler.io/man/bundle-config.1.html

### con bandera --local

Los docs dicen:
> xecuting `bundle config set --local <name> <value>` will set that configuration in the directory for the local application. The configuration will be stored in `<project_root>/.bundle/config`


### deployment

Se configura con:
```bash
bundle config set --local deployment true
```

No me queda claro:
> In deployment mode, Bundler will 'roll-out' the bundle for `production` use.

### path

Comando es:
```bash
bundle config set --local path '/home/ubuntu/PROYECTO/deployments/api-gems/bundle'
```

La ruta en disco donde las gemas del bundle serán instaladas. Ignora cualquier valor en `$GEM_PATH` o `$GEM_HOME`.

### without

Comando:
```bash
bundle config set --local without 'development test'
```

Lista separada por espacios o por `:` de los grupos de gemas a no ser instalados.
