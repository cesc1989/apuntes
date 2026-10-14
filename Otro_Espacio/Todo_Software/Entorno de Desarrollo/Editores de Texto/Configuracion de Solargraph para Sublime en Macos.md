# Configuración correcta de Solargraph para Sublime Text 4 en Macos

Esto es para corregir lo que está descrito en [[Apuntes_de_Sublime_Text#Problema con Solargraph luego de reiniciar el equipo]]

Pongo acá para que ese doc quede más sencillo.

## ✅ Solución que funcionó (2026-10-14)

Configuración en `Preferences > Package Settings > LSP > Server Configurations` (archivo `Packages/User/LanguageServers.sublime-settings`):

```json
{
    "ruby": {
        "enabled": true,
        "command": ["/Users/francisco/.gem/ruby/3.4.8/bin/bundle", "exec", "solargraph", "stdio"],
        "selector": "source.ruby",
        "initialization_options": {
            "diagnostics": true
        },
        "env": {
            "PATH": "/Users/francisco/.gem/ruby/3.4.8/bin:/Users/francisco/.rubies/ruby-3.4.8/bin:/opt/homebrew/bin:/usr/bin:/bin",
            "GEM_HOME": "/Users/francisco/.gem/ruby/3.4.8",
            "GEM_PATH": "/Users/francisco/.gem/ruby/3.4.8:/Users/francisco/.rubies/ruby-3.4.8/lib/ruby/gems/3.4.0"
        }
    }
}
```

#### Por qué fallaba antes

- La config de la documentación (`"command": ["solargraph", "stdio"]`) depende del `PATH` que tenga Sublime.
- Sublime abierto desde el Dock no carga `.bash_profile`, así que **chruby no se activa**. El ejecutable `solargraph` empieza con `#!/usr/bin/env ruby` y terminaba usando el **Ruby 2.6 del sistema**. De ahí venían los errores de sintaxis en `json` y el "crashed 5 times", que también le pasaba a ruby-lsp.
- Además, sin `bundle exec` se usa la versión de solargraph más nueva instalada globalmente y no la del `Gemfile.lock` del proyecto.

#### Qué hace cada parte

- `command` con la **ruta absoluta de `bundle`**: no depende del `PATH` de Sublime para encontrarlo.
- `bundle exec`: usa las versiones de `solargraph` y `solargraph-rails` del `Gemfile.lock` del proyecto.
- `env`: replica lo que haría chruby. Con `PATH` el `#!/usr/bin/env ruby` resuelve a Ruby 3.4.8, y con `GEM_HOME` y `GEM_PATH` RubyGems busca en las carpetas correctas.
  - Los valores salen de `env | grep -E "^(GEM_HOME|GEM_PATH)="` en una terminal dentro del proyecto.

#### Lo que NO funciona

- Envolver el comando en `bash -l -c "bundle exec solargraph stdio"`. Mi shell de login imprime secuencias de escape de la terminal (`\e[?2004l\e(B\e[m`) en stdout, y eso corrompe el protocolo LSP, que habla por stdout.

#### Cómo verificar

1. `LSP: Toggle Log Panel`: sin errores y sin rutas a `/System/Library/Frameworks/Ruby.framework`.
2. `LSP: Restart Servers` después de cambiar la config.
3. En un archivo de `app/`, autocompletar `expenditure.am…` ofrece `amount` (columna de `db/schema.rb`, gracias a `solargraph-rails`).
4. Una comilla simple a propósito muestra el diagnóstico de RuboCop.
5. La primera vez tarda varios minutos en indexar (en cashflow, ~5 min).

Desde la terminal, para confirmar qué Ruby y qué carpeta usa el proceso:

```bash
ps -axo pid,command | grep solargraph
lsof -a -p <PID> -d cwd          # debe ser la carpeta del proyecto
lsof -p <PID> | grep bin/ruby    # debe ser ~/.rubies/ruby-3.4.8/bin/ruby
```

#### Requisitos en el proyecto

- `solargraph` y `solargraph-rails` en el `Gemfile` (grupo `:development`), en versiones compatibles entre sí. En cashflow: **solargraph 0.58.3 + solargraph-rails 1.3**.
  - solargraph-rails 1.1.0 no funciona con solargraph 0.56.2: falla con `wrong number of arguments` y no carga las columnas.
  - solargraph 0.56.2 se cae con el RBS de `multi_xml` (`ComplexTypeError`).
- Un `.solargraph.yml` con `plugins: [solargraph-rails]`.

#### Otros proyectos con otra versión de Ruby

Esta config global queda fija en Ruby 3.4.8. Para un proyecto con otra versión, se sobrescribe la entrada `ruby` en su `.sublime-project`:

```json
{
    "folders": [{ "path": "." }],
    "settings": {
        "LSP": {
            "ruby": {
                "command": ["/Users/francisco/.gem/ruby/X.Y.Z/bin/bundle", "exec", "solargraph", "stdio"],
                "env": {
                    "PATH": "/Users/francisco/.gem/ruby/X.Y.Z/bin:/Users/francisco/.rubies/ruby-X.Y.Z/bin:/opt/homebrew/bin:/usr/bin:/bin",
                    "GEM_HOME": "/Users/francisco/.gem/ruby/X.Y.Z",
                    "GEM_PATH": "/Users/francisco/.gem/ruby/X.Y.Z:/Users/francisco/.rubies/ruby-X.Y.Z/lib/ruby/gems/X.Y.0"
                }
            }
        }
    }
}
```