# Migrando a Ruby 3.0.2 con WKHTMLTOPDF

Enlaces:

- Relacionado con respecto a Linux Alpine y wkhtmltopdf [+Apuntes de Docker - Parte 1](https://paper.dropbox.com/doc/Apuntes-de-Docker-Parte-1-PxjQ3IeKE8xn3T4iKDpCb) 
- Página de [versiones](https://www.ruby-lang.org/en/downloads/releases/) de Ruby
- Guía para [instalar Ruby en chip M1](https://www.moncefbelyamani.com/how-to-install-xcode-homebrew-git-rvm-ruby-on-mac/#step-2-install-chruby-and-the-latest-ruby-with-ruby-install) de mac

La dificultad en el proyecto Therapist estará con este paquete teniendo en cuenta que está solo en las versiones de Linux Alpine 3.14 e inferiores.

- [Alpine v3.12](https://pkgs.alpinelinux.org/packages?name=wkhtmltopdf&branch=v3.12&repo=&arch=&maintainer=)
- [Alpine v3.14](https://pkgs.alpinelinux.org/packages?name=wkhtmltopdf&branch=v3.14&repo=&arch=&maintainer=)
- [Alpine v3.15](https://pkgs.alpinelinux.org/packages?name=wkhtmltopdf&branch=v3.15&repo=&arch=&maintainer=)
    - En esta ya no está disponible.

Imágenes de Ruby 3.x con alguna de las versiones que sirven de Alpine.

**Con Alpine 3.14**

- [Ruby 3.1.2 con Alpine 3.14](https://hub.docker.com/_/ruby/tags?page=1&name=3.1.2-alpine3.14)
- [Ruby 3.1.1 con Alpine 3.14](https://hub.docker.com/_/ruby/tags?page=1&name=3.1.1-alpine3.14)
- [Ruby 3.0.4 con Alpine 3.14](https://hub.docker.com/_/ruby/tags?page=1&name=3.0.4-alpine3.14)
- [Ruby 3.0.3 con Alpine 3.14](https://hub.docker.com/_/ruby/tags?page=1&name=3.0.3-alpine3.14)

**Con Alpine 3.12**

- Ruby 3.0.1 con Alpine 3.12 - [link](https://hub.docker.com/_/ruby/tags?page=1&name=3.0.1-alpine3.12)
- Ruby 3.0.2 con Alpine 3.12 - [link](https://hub.docker.com/_/ruby/tags?page=1&name=3.0.2-alpine3.12)


# Resolución

Elegir la versión de Alpine 3.12 para Ruby 3.0.2


## Actualización #1

La versión de *wkhtmltopdf* disponible en Alpine 3.12 es **0.12.5-r1** y no genera los PDFs con estilos ni colores.

Subí a Ruby 3.0.2 Alpine 3.14 pero dio error al intentar instalar `ttf-ubuntu-font-family` porque este paquete no está en Alpine 3.14 sino hasta la versión 3.13

Probé instalando desde el repositorio de paquetes mediante comando `apk add` y parece funcionar.


# Sobre instalar Ruby en versiones anteriores específicas

Si la versión de Ruby es anterior a alguna de estas:

- 3.1.3
- 3.0.5
- 2.7.7

Toca instalarlo, por ejemplo la versión 3.1.0 (en un mac), así:

    ruby-install 3.1.0 -- --enable-shared

Visto en el tutorial de [Ruby on Mac](https://www.moncefbelyamani.com/how-to-install-xcode-homebrew-git-rvm-ruby-on-mac/#how-to-install-different-versions-of-ruby-and-switch-between-them).

