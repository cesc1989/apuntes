# Apuntes de Sublime Text

# Alto Consumo de CPU sin Hacer Nada

Cuando tenía el plugin de TypeScript y abría el proyecto de Luna Dashboard o DEV, Sublime empezaba a consumir hasta 90% de cpu, el MBP se recalentaba y parecía avión a punto de despegar.

Al parecer, se debe a [un asunto de indexación](https://stackoverflow.com/questions/50722309/sublime-text-3-using-massive-amount-of-cpu-on-idle#50722425) de archivos. ¿Solución? Esperar a que termine o desactivarla.

Se puede ver el estado de indexación en el menú Help → Indexing Status.

Se puede desactivar en Preferences.sublime-settings (Preference > Settings)

```json
"index_files": false
```

# Cómo abrir sublime mediante el comando subl en macos

Resulta que toca configurarlo primero:
```bash
francisco@Francisco-Quintero-M1 dotfiles % subl .
zsh: command not found: subl
```

Se configura así:
```bash
sudo ln -s /Applications/Sublime\ Text.app/Contents/SharedSupport/bin/subl /usr/local/bin/subl

francisco@Francisco-Quintero-M1 dotfiles % which subl
/usr/local/bin/subl
```

Visto en [Stack Overflow](https://stackoverflow.com/a/18842982/1407371).

# Cómo sincronizar paquetes y configuraciones

En primera medida usar Sync Settings.

Cuando falle sync settings ya que está sin mantenimiento:

- [Stack Overflow](https://stackoverflow.com/questions/29687417/is-there-a-way-to-sync-sublime-text-settings-across-multiple-computers)
- [Package Control docs](https://packagecontrol.io/docs/syncing)

En este [gist tengo todo](https://gist.github.com/cesc1989/27d65da13566a720f390a5e4b303595f) el último export.

La ruta donde encuentro las configuraciones en el macbook es:
```bash
# Macos Intel
cd ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/User

# Macos M1
cd ~/Library/Application\ Support/Sublime\ Text/Packages/User
```

Los archivos que salen en esa carpeta son:
```bash
~/Library/Application Support/Sublime Text 3/Packages/User
$ ls -la
total 40

-rw-r--r-- Default (OSX).sublime-keymap
-rw-r--r-- Elixir.sublime-settings
-rw-r--r-- Package Control.sublime-settings
-rw-r--r-- Package Control.user-ca-bundle
-rw-r--r-- Preferences.sublime-settings
```

Pero el gist que generaba Sync Settings tiene esto
```bash
-rw-r--r--@ Default%20%28Linux%29.sublime-keymap
-rw-r--r--@ Distraction%20Free.sublime-settings
-rw-r--r--@ Elixir.sublime-settings
-rw-r--r--@ GitGutter.sublime-settings
-rw-r--r--@ Go.sublime-settings
-rw-r--r--@ HTML%20%28Rails%29.sublime-settings
-rw-r--r--@ HTML.sublime-settings
-rw-r--r--@ JavaScript%20%28Babel%29.sublime-settings
-rw-r--r--@ JavaScript%20%28Rails%29.sublime-settings
-rw-r--r--@ JavaScript.sublime-settings
-rw-r--r--@ Package%20Control.sublime-settings
-rw-r--r--@ Preferences.sublime-settings
-rw-r--r--@ Ruby%20Slim.sublime-settings
-rw-r--r--@ Ruby.sublime-settings
-rw-r--r--@ SCSS.sublime-settings
```

## Pasos para sincronizar con git

Siguiendo los pasos de [esta respuesta](https://stackoverflow.com/a/38694182/1407371) en Stack Overflow.

En el mac anterior, crea un repo vacío en la carpeta de Sublime
```bash
$ cd ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/User

$ git init
```

y luego crea un archivo `.gitignore` con este contenido:
```
# Ignore everything...
*
# ... except preferences and package list
!.gitignore
!Preferences.sublime-settings
!Package Control.sublime-settings
```

Se ignora todo lo que esté ahí excepto las configuraciones de Sublime y de Package control.

Si queremos sincronizar otra cosa más, se agrega al `.gitignore` con el nombre del archivo precedido de signo de admiración que cierra.

Se agrega el repo creado en GitHub, se hace commit y luego push.

En el computador nuevo, se va a la carpeta de Sublime y se configura el repositorio:
```bash
cd ~/Library/Application\ Support/Sublime\ Text/Packages/User
git init
git remote add origin git@github.com:cesc1989/sublime-configs.git
git fetch
git reset --hard origin/main
```

Y luego hacer pull para traer solo los cambios del primer equipo. O configurar al gusto necesario.

Mi repo con las configuraciones → https://github.com/cesc1989/sublime-configs

## Más opciones para sincronización de configuraciones

Esta librería parece ayudar: https://www.alchemists.io/projects/sublime_text_kit

# Paquete desinstalado aparece en in_process_packages e ignored_packages

Me sale esto luego de haber desinstalado TypeScript Syntax.
```diff
diff --git a/Package Control.sublime-settings b/Package Control.sublime-settings
index 3316bd0..112a294 100644
--- a/Package Control.sublime-settings  
+++ b/Package Control.sublime-settings  
@@ -2,6 +2,7 @@
				"bootstrapped": true,
				"in_process_packages":
				[
+               "TypeScript Syntax",
				],
				"installed_packages":
				[
diff --git a/Preferences.sublime-settings b/Preferences.sublime-settings
index 8a4a671..3746049 100644
--- a/Preferences.sublime-settings
+++ b/Preferences.sublime-settings
@@ -1,7 +1,11 @@
+// This is Packages/User/Preferences.sublime-settings
+// and it's overriding the main Sublime preferences settings
+
 {
				"font_size": 12,
				"ignored_packages":
				[
+               "TypeScript Syntax",
								"Vintage",
				],
				"theme": "Default.sublime-theme",
```

Y hay un [error al respecto](https://github.com/wbond/package_control/issues/1640).

Quité el cambio en el archivo “Package Control.sublime-settings” e hice commit y mandé al repo. Todo parece seguir funcionando normalmente.

# 🌟 Potenciando Sublime Text 4 🌟

## Con Solargraph

Instalé la extensión de Solargraph para Sublime y ahora está más powa.

Aquí están [listados](https://github.com/castwide/solargraph#using-solargraph).

- Hay que instalar el paquete LSP
- La gema [solargraph](https://github.com/castwide/solargraph#installation)
- y configurar LSP como [indican](https://lsp.sublimetext.io/language_servers/#ruby-ruby-on-rails)

Está la gema [solargraph-rails](https://github.com/iftheshoefritz/solargraph-rails/) para mejor soporte para Ruby on Rails

### Problema con Solargraph luego de reiniciar el equipo

![error de solargraph](error.solargraph.png)

Enlaces:

- [Guía de configuración](https://solargraph.org/guides/configuration) de Solargraph

Aquí se [menciona](https://lsp.sublimetext.io/troubleshooting/#2-lsp-cannot-find-my-language-server-no-such-file-or-directory-xyz) sobre este error. La solución fue [actualizar el PATH](https://lsp.sublimetext.io/troubleshooting/#updating-the-path-used-by-lsp-servers) para que Sublime pueda encontrar los ejecutables que necesita.

For macOS and Linux you can extend the path like so:
```bash
export PATH="/usr/local/bin:$PATH"
```

### Si el path no funciona, revisa el panel log

La guía de [troubleshooting the LSP for Sublime Text](https://lsp.sublimetext.io/troubleshooting/#2-lsp-cannot-find-my-language-server-no-such-file-or-directory-xyz) dice que se abra el Log Panel para ver qué pasa.

Al abrirlo veo este error:
```bash
Ignoring strscan-3.1.0 because its extensions are not built. Try: gem pristine strscan --version 3.1.0
ruby: /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require': /Users/francisco/.gem/ruby/3.1.0/gems/json-2.9.1/lib/json/common.rb:154: syntax error, unexpected ..., expecting ')' (SyntaxError)
ruby:     def detailed_message(...)
ruby:                          ^~~
ruby: /Users/francisco/.gem/ruby/3.1.0/gems/json-2.9.1/lib/json/common.rb:164: dynamic constant assignment
ruby:   UnparserError = GeneratorError # :nodoc:
ruby:   ^~~~~~~~~~~~~
ruby: /Users/francisco/.gem/ruby/3.1.0/gems/json-2.9.1/lib/json/common.rb:168: class definition in method body
ruby:   class MissingUnicodeSupport < JSONError; end
ruby:   ^~~~~~~~~~~~~~~~~~~~~~~~~~~
ruby: /Users/francisco/.gem/ruby/3.1.0/gems/json-2.9.1/lib/json/common.rb:844: syntax error, unexpected end, expecting end-of-input
ruby: 	from /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
ruby: 	from /Users/francisco/.gem/ruby/3.1.0/gems/json-2.9.1/lib/json.rb:2:in `<top (required)>'
```

Al probar el comando que sugiere la gema parece instalar correctamente pero el error persiste:
```bash
$ gem pristine strscan --version 3.1.0
Restoring gems to pristine condition...
Building native extensions. This could take a while...
Restored strscan-3.1.0
```

### ¿Está usando la versión correcta de Ruby?

Al ver el log anterior parece que no está usando la versión correcta de Ruby sino la que viene por defecto en el sistema (esta no hay que usarla nunca):
```bash
ruby: /System/Library/Frameworks/Ruby.framework/Versions/2.6/usr/lib/ruby/2.6.0/rubygems/core_ext/kernel_require.rb:54:in `require'
```

## Con Ruby LSP de Shopify

Decidí cambiar de gema. Desinstalé todo lo de solargraph porque igual no me está funcionando y me fio con ruby-lsp pero obtengo el mismo error.

Enlaces:
- [Ruby LSP](https://shopify.github.io/ruby-lsp/editors.html)
- Configuración con [Sublime LSP](https://lsp.sublimetext.io/language_servers/#ruby-lsp)

Sobre el último error que obtuve, en [este issue](https://github.com/Shopify/ruby-lsp/issues/2842#issuecomment-2484052337) arrojan algo de luz. Puede ser con algo de chruby no cargando en el mismo entorno que Sublime Text. Por eso es que se ve que intenta cargar la versión del sistema.

# Ocultar carpeta de resultados de búsqueda

En los proyectos Coshi Notes y Cashflow la carpeta `tmp/` tiene muchos archivos que aparecen al hacer búsquedas o la combinación CDM + P.

Para ocultarlos hay dos formas:

- "folder_exclude_patterns": ["tmp"]
- "binary_file_patterns": ["tmp/"]

`folder_exclude_patterns` esta funciona bien pero también oculta la carpeta del sidebar.

En cambio `binary_file_patterns` los oculta solo de los resultados de búsqueda y deja la carpeta visible en el sidebar.

Visto en [Stack Overflow](https://stackoverflow.com/a/55748485/1407371).

# No abrir ventana anterior en Linux Mint

En Linux Mint, cuando abro una carpeta con el comando desde la consola `subl .`, se abre la ventana anterior junto con la actual. Eso es incomodo y no me gusta.

Para corregirlo hay que agregar estas configuraciones en el archivo de preferencias de usuario:
```json
{
  "hot_exit": false,
  "remember_open_files": false
}
```

Visto en:
- [Stack Overflow](https://stackoverflow.com/questions/12193913/when-opening-a-directory-through-command-line-sublime-text-opens-two-windows-in)