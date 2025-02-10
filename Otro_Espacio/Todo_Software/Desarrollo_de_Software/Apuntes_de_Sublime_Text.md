# Apuntes de Sublime Text

# Alto Consumo de CPU sin Hacer Nada

Cuando tenía el plugin de TypeScript y abría el proyecto de Luna Dashboard o DEV, Sublime empezaba a consumir hasta 90% de cpu, el MBP se recalentaba y parecía avión a punto de despegar.

Al parecer, se debe a [un asunto de indexación](https://stackoverflow.com/questions/50722309/sublime-text-3-using-massive-amount-of-cpu-on-idle#50722425) de archivos. ¿Solución? Esperar a que termine o desactivarla.

Se puede ver el estado de indexación en el menú Help → Indexing Status.

Se puede desactivar en Preferences.sublime-settings (Preference > Settings)

    "index_files": false


# Cómo abrir sublime mediante el comando subl en macos

Resulta que toca configurarlo primero:

    francisco@Francisco-Quintero-M1 dotfiles % subl .
    zsh: command not found: subl

Se configura así:

    sudo ln -s /Applications/Sublime\ Text.app/Contents/SharedSupport/bin/subl /usr/local/bin/subl
    
    francisco@Francisco-Quintero-M1 dotfiles % which subl
    /usr/local/bin/subl

Visto en [Stack Overflow](https://stackoverflow.com/a/18842982/1407371).


# Cómo sincronizar paquetes y configuraciones

En primera medida usar Sync Settings.

Cuando falle sync settings ya que está sin mantenimiento:

- [Stack Overflow](https://stackoverflow.com/questions/29687417/is-there-a-way-to-sync-sublime-text-settings-across-multiple-computers)
- [Package Control docs](https://packagecontrol.io/docs/syncing)

En este [gist tengo todo](https://gist.github.com/cesc1989/27d65da13566a720f390a5e4b303595f) el último export.

La ruta donde encuentro las configuraciones en el macbook es:

    # Macos Intel
    cd ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/User
    
    # Macos M1
    cd ~/Library/Application\ Support/Sublime\ Text/Packages/User

Los archivos que salen en esa carpeta son:

    ~/Library/Application Support/Sublime Text 3/Packages/User
    $ ls -la
    total 40
    
    -rw-r--r-- Default (OSX).sublime-keymap
    -rw-r--r-- Elixir.sublime-settings
    -rw-r--r-- Package Control.sublime-settings
    -rw-r--r-- Package Control.user-ca-bundle
    -rw-r--r-- Preferences.sublime-settings

Pero el gist que generaba Sync Settings tiene esto

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


## Pasos para sincronizar con git

Siguiendo los pasos de [esta respuesta](https://stackoverflow.com/a/38694182/1407371) en Stack Overflow.

En el mac anterior, crea un repo vacío en la carpeta de Sublime

    $ cd ~/Library/Application\ Support/Sublime\ Text\ 3/Packages/User
    
    $ git init

y luego crea un archivo `.gitignore` con este contenido:

    # Ignore everything...
    *
    # ... except preferences and package list
    !.gitignore
    !Preferences.sublime-settings
    !Package Control.sublime-settings

Se ignora todo lo que esté ahí excepto las configuraciones de Sublime y de Package control.

Si queremos sincronizar otra cosa más, se agrega al `.gitignore` con el nombre del archivo precedido de signo de admiración que cierra.

Se agrega el repo creado en GitHub, se hace commit y luego push.

En el computador nuevo, se va a la carpeta de Sublime y se configura el repositorio:

    $ cd ~/Library/Application\ Support/Sublime\ Text/Packages/User
    $ git init
    $ git remote add origin git@github.com:cesc1989/sublime-configs.git
    $ git fetch
    $ git reset --hard origin/main

Y luego hacer pull para traer solo los cambios del primer equipo. O configurar al gusto necesario.

Mi repo con las configuraciones → https://github.com/cesc1989/sublime-configs


## Más opciones para sincronización de configuraciones

Esta librería parece ayudar: https://www.alchemists.io/projects/sublime_text_kit


# Paquete desinstalado aparece en in_process_packages e ignored_packages

Me sale esto luego de haber desinstalado TypeScript Syntax.

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

Y hay un [error al respecto](https://github.com/wbond/package_control/issues/1640).

Quité el cambio en el archivo “Package Control.sublime-settings” e hice commit y mandé al repo. Todo parece seguir funcionando normalmente.


# Potenciando Sublime Text 4

## Paquetes

Instalé la extensión de Solargraph para Sublime y ahora está más powa.

Aquí están [listados](https://github.com/castwide/solargraph#using-solargraph).

- Hay que instalar el paquete LSP
- La gema [solargraph](https://github.com/castwide/solargraph#installation)
- y configurar LSP como [indican](https://lsp.sublimetext.io/language_servers/#ruby-ruby-on-rails)

Está la gema [solargraph-rails](https://github.com/iftheshoefritz/solargraph-rails/) para mejor soporte para Ruby on Rails

## Problema con solargraph luego de reiniciar el equipo

![](https://paper-attachments.dropboxusercontent.com/s_0AD64D6B273F865C1BB511C91349B747E2010EA1B6271BE0DC7EAFE6834C7018_1694635900242_error.solargraph.png)


Enlaces:

- [Guía de configuración](https://solargraph.org/guides/configuration) de Solargraph

Aquí se [menciona](https://lsp.sublimetext.io/troubleshooting/#2-lsp-cannot-find-my-language-server-no-such-file-or-directory-xyz) sobre este error. La solución fue [actualizar el PATH](https://lsp.sublimetext.io/troubleshooting/#updating-the-path-used-by-lsp-servers) para que Sublime pueda encontrar los ejecutables que necesita.

For macOS and Linux you can extend the path like so:

    export PATH="/usr/local/bin:$PATH"

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