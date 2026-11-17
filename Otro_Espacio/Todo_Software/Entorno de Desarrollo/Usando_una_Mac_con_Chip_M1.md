# Usando una Mac con Chip M1

Etiquetas: #apple_silicon #mac_m1

Incidencias de lo que me pasó al cambiar a una mac con chip M1.

Enlaces:

- En [esta guía](https://www.moncefbelyamani.com/how-to-install-xcode-homebrew-git-rvm-ruby-on-mac/#start-here-if-you-choose-the-long-and-manual-route) fue que descubrí la forma de instalar Ruby en Mac chip M1

# Error al instalar la gema sass-rails

    Fetching sassc 2.2.1
    Installing sassc 2.2.1 with native extensions
    Gem::Ext::BuildError: ERROR: Failed to build gem native extension.
    
        current directory: /Users/francisco/.gem/ruby/2.7.7/gems/sassc-2.2.1/ext
    /Users/francisco/.rubies/ruby-2.7.7/bin/ruby -I /Users/francisco/.rubies/ruby-2.7.7/lib/ruby/2.7.0 -r ./siteconf20230219-12465-11m5bzy.rb
    extconf.rb
    creating Makefile
    
    current directory: /Users/francisco/.gem/ruby/2.7.7/gems/sassc-2.2.1/ext
    make "DESTDIR=" clean
    
    current directory: /Users/francisco/.gem/ruby/2.7.7/gems/sassc-2.2.1/ext
    make "DESTDIR="
    compiling ./libsass/src/units.cpp
    clang: error: the clang compiler does not support '-march=native'
    make: *** [units.o] Error 1
    
    make failed, exit code 2
    
    Gem files will remain installed in /Users/francisco/.gem/ruby/2.7.7/gems/sassc-2.2.1 for inspection.
    Results logged to /Users/francisco/.gem/ruby/2.7.7/extensions/arm64-darwin-22/2.7.0-static/sassc-2.2.1/gem_make.out
    
    An error occurred while installing sassc (2.2.1), and Bundler cannot continue.
    Make sure that `gem install sassc -v '2.2.1' --source 'https://rubygems.org/'` succeeds before bundling.
    
    In Gemfile:
      sass-rails was resolved to 6.0.0, which depends on
        sassc-rails was resolved to 2.1.2, which depends on
          sassc


> Una de las formas era desinstalar esta gema ya que no se está usando en el proyecto Patient Forms.

Cuando intenté correr el comando para instalar la gema:

    $ gem install sassc -v '2.2.1' --source 'https://rubygems.org/01'
    Building native extensions. This could take a while...
    ERROR:  Error installing sassc:
            ERROR: Failed to build gem native extension.
    
        current directory: /Users/francisco/.gem/ruby/2.7.7/gems/sassc-2.2.1/ext
    /Users/francisco/.rubies/ruby-2.7.7/bin/ruby -I /Users/francisco/.rubies/ruby-2.7.7/lib/ruby/2.7.0 -r ./siteconf20230219-15017-iciyw8.rb extconf.rb
    creating Makefile
    
    current directory: /Users/francisco/.gem/ruby/2.7.7/gems/sassc-2.2.1/ext
    make "DESTDIR=" clean
    
    current directory: /Users/francisco/.gem/ruby/2.7.7/gems/sassc-2.2.1/ext
    make "DESTDIR="
    compiling ./libsass/src/units.cpp
    clang: error: the clang compiler does not support '-march=native'
    make: *** [units.o] Error 1
    
    make failed, exit code 2
    
    Gem files will remain installed in /Users/francisco/.gem/ruby/2.7.7/gems/sassc-2.2.1 for inspection.
    Results logged to /Users/francisco/.gem/ruby/2.7.7/extensions/arm64-darwin-22/2.7.0-static/sassc-2.2.1/gem_make.out

**La forma en que se instaló fue:**

    $ gem install sassc -- --disable-march-tune-native
    Building native extensions with: '--disable-march-tune-native'
    This could take a while...
    Successfully installed sassc-2.4.0
    Parsing documentation for sassc-2.4.0
    Installing ri documentation for sassc-2.4.0
    Done installing documentation for sassc after 0 seconds
    1 gem installed

Pero seguía fallando el comando bundle. Probé poniendo la versión indicada:

    $ gem install sassc -v '2.2.1' -- --disable-march-tune-native

Y luego el bundle fue exitoso.

Solución vista en [Github Issue](https://github.com/sass/sassc-ruby/issues/222#issuecomment-857092485).

# Bracketed Paste mode

Cada vez que pego algo en la terminal se agregan unos caracteres adicionales:

    $ 00~rake db:hoos_and_koos:run_all01~

Que fastidio. Según se ve en algunos posts es la locura pero no veo porque.

Para desactivarlo en Bash no hay una variable que se pueda colocar en algún archivo.

Sin embargo, con esta instrucción se puede desactivar. La agregué a mis dotfiles en el archivo `.prompt`

    printf '\e[?2004l'

para que funcione hay que cargar el archivo `.prompt` en el archivo `.bashrc` y asegurarme que se cargue después de cargar los dotfiles.

Así:

```bash
export REDIS_URL=redis://localhost:6379

if [ -f "$HOME/.bash_profile" ]; then
  . "$HOME/.bash_profile"
fi

# Include personal dotfiles prompt_git function into subshells.
[ -f "$HOME/projects/dotfiles/system/.prompt" ] && source $HOME/projects/dotfiles/system/.prompt
```

Enlaces:

- [En Ask Different](https://apple.stackexchange.com/questions/391940/copying-and-pasting-between-terminal-windows-adds-extra-characters)
- Conrad Irwin → https://cirw.in/blog/bracketed-paste
    - Aquí se dan más detalles detrás de esto y cómo activar y desactivarlo
- Bracketed Paste Mode in Terminal → https://jdhao.github.io/2021/02/01/bracketed_paste_mode/
- How do I disable the weird characters from "bracketed paste mode" on the Mac OS X default terminal? → https://stackoverflow.com/questions/42212099/how-do-i-disable-the-weird-characters-from-bracketed-paste-mode-on-the-mac-os/50654284#50654284


# Instalar Python 2 para error de node-sass

Cuando ejecuté el comando `yarn install` para el proyecto dashboard encontré este error:

    error /Users/francisco/projects/luna-project/clinical-dashboard-backend/node_modules/node-sass: Command failed.
    Exit code: 1
    Command: node scripts/build.js
    Arguments: 
    Directory: /Users/francisco/projects/luna-project/clinical-dashboard-backend/node_modules/node-sass
    Output:
    Building: /Users/francisco/.nvm/versions/node/v14.17.3/bin/node /Users/francisco/projects/luna-project/clinical-dashboard-backend/node_modules/node-gyp/bin/node-gyp.js rebuild --verbose --libsass_ext= --libsass_cflags= --libsass_ldflags= --libsass_library=
    gyp info it worked if it ends with ok
    gyp verb cli [
    gyp verb cli   '/Users/francisco/.nvm/versions/node/v14.17.3/bin/node',
    gyp verb cli   '/Users/francisco/projects/luna-project/clinical-dashboard-backend/node_modules/node-gyp/bin/node-gyp.js',
    gyp verb cli   'rebuild',
    gyp verb cli   '--verbose',
    gyp verb cli   '--libsass_ext=',
    gyp verb cli   '--libsass_cflags=',
    gyp verb cli   '--libsass_ldflags=',
    gyp verb cli   '--libsass_library='
    gyp verb cli ]
    gyp info using node-gyp@3.8.0
    gyp info using node@14.17.3 | darwin | arm64
    gyp verb command rebuild []
    gyp verb command clean []
    gyp verb clean removing "build" directory
    gyp verb command configure []
    gyp verb check python checking for Python executable "python2" in the PATH
    gyp verb `which` failed Error: not found: python2
    gyp verb `which` failed     at getNotFoundError (/Users/francisco/projects/luna-project/clinical-dashboard-backend/node_modules/which/which.js:13:12)

node-sass falló en instalar pero porque en Macos Ventura (v13) ya no se incluye Python 2.

Tocó instalar Python 2 con [Pyenv como mencionan](https://stackoverflow.com/a/71620699/1407371) aquí.


> [Pyenv](https://github.com/pyenv/pyenv) es el Rbenv de Python. Un gestor de version de Python.

Comandos:

    brew install pyenv
    pyenv install 2.7.18

Luego:

    pyenv global 2.7.18

Y luego agregué al PATH desde mis dotfiles:

    export PATH="$(pyenv root)/shims:${PATH}"


# Teclado externo Logitech no reconoce teclas < y >

Pues resulta que no podía usar las teclas menor que y mayor que aunque usará el layout Latin America o Spanish Legacy.

También probé Karabiner pero no funcionó. Lo que sí sirvió fue usar [este repo](https://github.com/neosergio/Latam-Keyboard) y sus configuraciones.

El comando final es:

```
git clone https://github.com/neosergio/Latam-Keyboard.git && cd Latam-Keyboard && cp -v Latam*.* ~/Library/Keyboard\ Layouts/
```

Selecciono el idioma de “Otras” fuentes en el “Input Sources” y al fin funcionan esas teclas con el teclado externo.

![](https://paper-attachments.dropboxusercontent.com/s_F5C527EAE87EDDFD6AA7A7B2D6D1C3FD50312C3AE5B0C463E952CDDDB1579B5E_1691596916481_Screenshot+2023-08-09+at+11.01.29+AM.png)


Con el teclado del macbook toca invertir los idiomas porque al teclear la tecla en cuestión queda invertida:
 
```
< = |
    
> = °
```

Pero mientras use el teclado externo estaremos bien con ese idioma.

Visto en [Superuser](https://superuser.com/a/1759650/372807).

