# Apuntes de Linux - Parte 2

# Comandos y Herramientas

- The `command` utility shall cause the shell to treat the arguments as a simple command → https://pubs.opengroup.org/onlinepubs/9699919799/utilities/command.html

# Cómo Actualizar AWS CLI con pip

Si se tiene Python 2, hay que usar pip. Si se tiene Python 3, hay que usar pip3.
```bash
$ python --version
Python 2.7.16

$ pip install --upgrade --user awscli

$ python --version
Python 3

$ pip3 install --upgrade --user awscli
```

La bandera `--user` le indica a pip que instale en el directorio del usuario y no en root.

# Comando `find` una vez más al rescate

Necesitaba copiar las capturas de la Switch las cuales están en una estructura de carpetas muy estricta: año/mes/día/captura.jpg

Con este comando, pude copiar, recursivamente, todos los archivos a otra carpeta:
```bash
$ find . -type f -exec cp {} ~/Pictures/capturas-switch/ \;
```

# Cómo cambiar la shell por defecto en Macos Ventura

En IW me dieron una macos nueva y vino con Ventura. En esta versión la shell por defecto es zsh. Quería cambiarala a bash para que funcionaran mis doftiles.

Se hace así:
```bash
chsh -s /bin/bash
```

y se abre y cierra de nuevo la terminal.

Visto en [cyberciti](https://www.cyberciti.biz/faq/change-default-shell-to-bash-on-macos-catalina/).

# Detalle con gnugp, gpg y gpg2 para instalar RVM

Tengo instalado gnugp o gpg en la versión dos pero según RVM necesito tener gpg2.
```bash
$ which gpg
/opt/homebrew/bin/gpg
```

Si miramos la versión:
```bash
$ gpg --version
gpg (GnuPG) 2.4.0
libgcrypt 1.10.1
Copyright (C) 2021 Free Software Foundation, Inc.
License GNU GPL-3.0-or-later <https://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Home: /Users/francisco/.gnupg
Supported algorithms:
Pubkey: RSA, ELG, DSA, ECDH, ECDSA, EDDSA
Cipher: IDEA, 3DES, CAST5, BLOWFISH, AES, AES192, AES256, TWOFISH,
		CAMELLIA128, CAMELLIA192, CAMELLIA256
Hash: SHA1, RIPEMD160, SHA256, SHA384, SHA512, SHA224
Compression: Uncompressed, ZIP, ZLIB, BZIP2
```

Encuentro que una solución es hacer un enlace simbolico:
```bash
ln -s /opt/homebrew/bin/gpg /opt/homebrew/bin/gpg2
```

Una vez hecho eso, se puede instalar RVM
```bash
$ gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
    
    gpg: key 105BD0E739499BDB: 1 duplicate signature removed
    gpg: key 105BD0E739499BDB: public key "Piotr Kuczynski <piotr.kuczynski@gmail.com>" imported
    gpg: key 3804BB82D39DC0E3: public key "Michal Papis (RVM signing) <mpapis@gmail.com>" imported
    gpg: Total number processed: 2
    gpg:               imported: 2
```

Visto en [Stack Overflow](https://stackoverflow.com/a/69920288/1407371).

# Instalar Ruby 2.7.7 en Mac con chip M1

Etiquetas: #ruby_old #ruby-install

En [esta guía](https://www.moncefbelyamani.com/how-to-install-xcode-homebrew-git-rvm-ruby-on-mac/#how-to-install-different-versions-of-ruby-and-switch-between-them) se recomienda usar chruby y ruby-install.

Para verificar que la terminal esté corriendo en modo M1:
```bash
$ uname -m
arm64
```

Si dice `x86_4`, está en modo Rosetta.

Para instalar [chruby](https://github.com/postmodern/chruby) y [ruby-install](https://github.com/postmodern/ruby-install).

    brew install chruby ruby-install

Luego para instalar una versión de Ruby:

    ruby-install 2.7.7

Para que cargue por defecto en todas las ventanas de terminal (en vez de la versión por defecto de Mac):

    echo "chruby ruby-2.7.7" >> ~/.bash_profile


> NOTA: en los chip M1 ya no es tan fácil instalar la versión 2.7.1
> 
> La recomendación del autor es que se actualicen los proyectos a usar una versión reciente de Ruby.
> 
> Tratar de instalar una versión más antigua es más complicado.

Para instalar versiones anteriores a 3.1.3, 3.0.5, or 2.7.7

Se revisa que valores salen en CLT o Xcode al ejecutar:
```bash
$ brew config
HOMEBREW_VERSION: 4.0.3
ORIGIN: https://github.com/Homebrew/brew
# (...)
macOS: 13.2.1-arm64
CLT: 14.2.0.0.1.1668646533
Xcode: N/A
Rosetta 2: false
```

En la guía dice que si empieza por 14 alguno de los dos, se instala la versión anterior de ruby así:
```bash
ruby-install 3.0.2 -- --enable-shared
```

## ¿cómo listar las versiones instaladas localmente al instalar con ruby-install?

El comando `ruby-install` sin argumentos solo devuelve las versiones disponibles para instalar.

Para poder listar las que se tiene en local hay que usar el comando `chruby` sin argumentos.

ruby-install sin argumentos:
```bash
$ ruby-install
Stable ruby versions:
  ruby:
	2.6.10
	2.7.8
	3.0.6
	3.1.4
	3.2.2
  jruby:
	9.4.2.0
  truffleruby:
	22.3.1
  truffleruby-graalvm:
	22.3.1
  mruby:
	3.0.0
```

chruby sin argumentos:
```bash
$ chruby
   ruby-2.7.7
   ruby-3.0.2
 * ruby-3.0.6
```

Si comparamos con lo que hay en la carpeta de instalación:
```bash
$ which ruby
/Users/francisco/.rubies/ruby-3.0.6/bin/ruby

$ ls /Users/francisco/.rubies/
ruby-2.7.7        ruby-3.0.2        ruby-3.0.6
```

Y esa es la forma correcta. Raro, no?

# Sobre Alpine Linux y sus versiones

Primero que nada, Alpine Linux no mantiene en almacenamiento el repositorio de versiones muy anteriores.

Así lo explican en el [issue tracker](https://gitlab.alpinelinux.org/alpine/abuild/-/issues/9996):

> You seem to have different expectation what the current policy is. We don't at the moment have resources to store all built packages indefinitely in our infra. Thus we currently keep only the latest for each stable branch, and has always been like that.

Entonces, ocurría que para usar Ruby 2.7.7 en la nube, la versión disponible de la imagen de contenedor era `ruby:2.7.7-alpine3.16` y en la versión 3.16 no estaba disponible el programa WKHTMLTOPDF.

Dicho programa solo está disponible hasta la versión 3.14 de Alpine Linux. Así que toca hacer un build propio o buscar otra forma de instalarlo.

Esto toca tenerlo en cuenta para cuando se suban las versiones de Ruby en proyectos que usen ese programa porque Alpine va dejando de dar soporte y a la vez eliminando paquetes de los respositorios.

Enlaces:
- En [este artículo](https://dev.to/asyazwan/moving-away-from-alpine-30n4) el autor menciona este problema.
- Página de [versiones](https://www.alpinelinux.org/releases/) de Alpine

# Instalación fallida de Ruby 3.0.4 con ruby-install

Concretamente este error:

    !!! Compiling ruby 3.0.4 failed!

Según [este tutorial](https://www.moncefbelyamani.com/how-to-install-xcode-homebrew-git-rvm-ruby-on-mac/#step-2-install-chruby-and-the-latest-ruby-with-ruby-install), la razón de la falla es que son versiones antiguas con poco soporte.

> You’re trying to install a version of Ruby other than 3.2.0+, 3.1.3+, 3.0.5+, or 2.7.7+

La solución la da en [esta sección](https://www.moncefbelyamani.com/how-to-install-xcode-homebrew-git-rvm-ruby-on-mac/#how-to-install-different-versions-of-ruby-and-switch-between-them) del mismo tutorial. Es la misma que para instalar Ruby 2.7.7

    ruby-install 3.0.4 -- --enable-shared

# Cómo listar elementos únicos de un archivo de texto

Para listar todos los elementos que NO se repiten en un archivo:
```bash
sort file.txt | uniq
```

Y para listar los elementos DUPLICADOS:
```bash
sort file.txt | uniq -d
```

La bandera `-d` significa:
```bash
-d, --repeated
     Output a single copy of each line that is repeated in the input.
```

Fuente: [SO](https://stackoverflow.com/a/12743847/1407371).

# Formato e interpolación de fechas en bash

## Interpolación en strings

¿Cómo interpolar fechas en una cadena? Veamos este ejemplo:
```bash
git commit -m "Database $(date '+%a %M:%H %h %d %Y')"
```

La clave es que hay que encerrar la expresión `$()` entre comilla dobles y el argumento de la función en comilla simple.

Enlaces
- [Stack Overflow](https://stackoverflow.com/questions/23655580/in-bash-how-do-i-interpolate-in-a-string)

## Formateo de fecha

Para darle un formato particular a una fecha, ejemplo, `YYYY-MM-DD`.

```bash
$(date +%F %T)
```

Revisa `man date` para más detalles.

Enlaces
- [Stack Overflow](https://stackoverflow.com/questions/1401482/yyyy-mm-dd-format-date-in-shell-script)

# Continue comando en Bash en la siguiente línea

O comandos multi línea.

Básicamente es poder decirle al script que, al ser una línea tan extensa, quiero que la siguiente esté en la siguiente línea. Ejemplo, esto en un Dockerfile:

```bash
RUN apk -Uuv add groff less python3 py3-pip \
    && pip install awscli \
    && apk --purge -v del py-pip \
    && rm /var/cache/apk/*
```

La clave es el backslash.

Enlaces:
- [Stack Overflow](https://stackoverflow.com/questions/7316107/bash-continuation-lines)
- [NixCraft](https://www.cyberciti.biz/faq/howto-ask-bash-that-line-command-script-continues-next-line/)

# ¿Cómo asignar la salida de un comando a una variable?

Si se hace así:
```bash
thefile= ls -t -U | grep -m 1 "Screen Shot";
```

No resulta por el espacio. Hay que quitar el espacio entre la asignación y el comando, además, debe usar substitución de comando:
```bash
thefile=$(ls -t -U | grep -m 1 "Screen Shot")
```

Otro ejemplo. Script:
```bash
#!/bin/bash

sublime_path=$(which subl)

echo "Path de sublime es: " + $sublime_path
```

Ejecución y salida:
```bash
$ bash prueba_command_subs.sh 

Path de sublime es:  + /usr/local/bin/subl
```

Enlaces:
- [Unix & Linux](https://unix.stackexchange.com/questions/16024/how-can-i-assign-the-output-of-a-command-to-a-shell-variable)
- [Command substitution](http://tldp.org/LDP/abs/html/commandsub.html)