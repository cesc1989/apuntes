# ADR-004 - Instala y usa Ruby con Mise

## mise: precompilados de Ruby

Ayer, 18 de Noviembre, se publicó esta nota de seguridad de Ruby: https://www.ruby-lang.org/en/news/2026/11/18/ruby-3-4-13-released/

La exposición es grave en el sentido de que hay posibilidad de ejecución remota. Por esto decidí aprovechar el momento para hacer varias cosas:

- **Empezar a usar mise**: su atractivo de binarios precompilados es muy interesante y útil para las instalaciones en local y en los VPS.
- **Subir a Ruby 4.0.x**: para estar más cerca de las versiones recientes y más actualizadas.
- **Mantenerme al día en seguridad**: en esta era de IA que saca vulnerabilidades a cada rato hay que estar en la juega.

Así que procederé a cambiar chruby por mise en todas las máquinas. Locales y remotas. Esto hay que hacerlo con cuidado porque no es un cambio sencillo. Por lo tanto iré paso a paso así:


MBP
- [x] Instalar y configurar
- [ ] Usar mise en el MBP y que los proyectos corran normalmente
- [ ] Eliminar chruby, rubies y ruby-install del MBP

dap_node
- [ ] Instalar y configurar
- [ ] Despliegue exitoso de cashflow
- [ ] Despliegue exitoso de coshinotes
- [ ] Despliegue exitoso de enlacito
- [ ] Eliminar chruby, rubies y ruby-install de dap_node

dap_hatch
- [ ] Instalar y configurar
- [ ] Despliegue exitoso de supermenu
- [ ] Despliegue exitoso de postlane
- [ ] Eliminar chruby, rubies y ruby-install de dap_hatch


## Instalación y Configuración

mise se instala con este script:
```bash
curl -fsSL https://mise.run | sh
```

El executable queda en la carpeta `~/.local/bin/mise`.

> [!Note]
> La activación la pude completar colocando el script en mi archivo `.bash_profile` y no en `.bashrc`.
 
Para la activación hay que agregar esta línea al `.bash_profile`
```bash
eval "$(/Users/francisco/.local/bin/mise activate bash)"
```

### mise doctor

Este comando sirve para confirmar que la instalación fue correcta. Cuando hay cosas mal puestas saca una salida como:
```bash
5 problems found:

1. mise is not activated, run mise help activate or
   read documentation at https://mise.jdx.dev for activation instructions.
   Alternatively, add the shims directory ~/.local/share/mise/shims to PATH.
   Using the shims directory is preferred for non-interactive setups.

2. tool core:elixir@1.15.4-otp-26 is not installed, install with `mise install`

3. tool core:erlang@26.0.2 is not installed, install with `mise install`

4. tool core:node@22.14.0 is not installed, install with `mise install`

5. tool aqua:direnv/direnv@2.37.1 is not installed, install with `mise install`
```

### Desactiva chruby

No pueden convivir en un mismo sistema. Por eso es mejor desactivar chruby antes de empezar a usar mise.

Para los equipos locales toca quitarlo del `.bash_profile` en dotfiles.

Para los VPS toca quitarlo del `.profile` y de los scripts de despliegue en el paso 003.

### Verificación que sistema usa Ruby de mise

Para confirmar que el binario de ruby que se carga es el de mise se puede correr el comando `type -a ruby`.

Salida de ejemplo:
```bash
type -a ruby
ruby is /Users/francisco/.local/share/mise/installs/ruby/3.4.13/bin/ruby
ruby is /Users/francisco/.local/share/mise/shims/ruby
ruby is /usr/bin/ruby
ruby is /Users/francisco/.rubies/ruby-3.4.8/bin/ruby
```

Cuando todo está bien configurado para mise aparece de primero.

También el comando `gem env home` debería apuntar a:
```bash
~/.local/share/mise/installs/ruby/3.4.13/
```


### Warnings por mezcla con chruby

Antes de haber desactivado chruby corrí las pruebas y me salían estos mensajes:
```bash
Source locally installed gems is ignoring #<Bundler::StubSpecification name=websocket-driver version=0.7.6 platform=ruby> because it is missing extensions
```

Para quitarlos hay que correr el comando:
```bash
chruby_reset && hash -r
```

### Configuración

De momento estas dos opciones las quiero en el archivo de configuraciones:
```toml
[settings]
idiomatic_version_file_enable_tools = ["ruby"]
[settings.ruby]
compile = false
```

La primera es para que lea el archivo `.ruby-version` de cada repo.

La segunda es para que siempre use binarios precompilados.

### Carpetas

Las versiones de los tools quedan instaladas en `~/.local/share/mise/installs/`.

## Uso e instalación de rubies

Para instalar una versión de ruby:
```bash
mise use ruby@3.4.8
```

Escribe al archivo `mise.toml` e instala esa versión de Ruby.