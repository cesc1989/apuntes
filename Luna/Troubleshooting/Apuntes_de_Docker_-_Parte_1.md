# Apuntes de Docker - Parte 1

# Error con Docker desktop
## `Error response from daemon: Get https://registry-1.docker.io/v2/: unauthorized: incorrect username or password`

Me ocurrió cuando traté de crear la imagen de la aplicación frontend de Therapist Forms.

Temporalmente solucionado al hacer `docker logout` cómo comentan en [este Issue](https://github.com/docker/hub-feedback/issues/1098#issuecomment-316309768).


# Error de Linux Alpine que no tiene wkhtmltopdf

Enlaces:

- Página de [listado de versiones](https://www.ruby-lang.org/en/downloads/releases/) liberadas de Ruby
- Ruby 2.7.7 con Alpine 3.16 en [Docker Hub](https://hub.docker.com/_/ruby/tags?page=1&name=2.7.7)
- Página para [listar los paquetes](https://pkgs.alpinelinux.org/packages?name=wkhtmltopdf&branch=v3.14&repo=&arch=&maintainer=) disponibles según la versión de Alpine
    - [Acá el paquete](https://pkgs.alpinelinux.org/packages?name=ttf-ubuntu-font-family&branch=v3.12&repo=&arch=&maintainer=) ttf-ubuntu-font-family
- Alpine 3.14 con wkhtmltopdf (0.12.6) - [Ver](https://pkgs.alpinelinux.org/packages?name=wkhtmltopdf&branch=v3.14&repo=&arch=&maintainer=).
    - Última versión de Alpine donde está disponible wkhtmltopdf.
- Pregunta al [respecto](https://stackoverflow.com/questions/59026248/how-to-install-wkhtmltopdf-on-docker-ruby-2-5-1-alpine-linux) en Stack Overflow

Cuando intenté moverme a Ruby 2.7.7 configuré la imagen de Alpine a la versión 3.16 pero resulta que esa versión de alpine no tiene disponible el paquete wkhtmltopdf.

    + apk add --no-cache --virtual .app-rundeps postgresql-dev tzdata nodejs wkhtmltopdf libxrender libxext gcompat fontconfig freetype ttf-dejavu ttf-droid ttf-freefont ttf-liberation ttf-ubuntu-font-family imagemagick
    fetch https://dl-cdn.alpinelinux.org/alpine/v3.16/main/x86_64/APKINDEX.tar.gz
    fetch https://dl-cdn.alpinelinux.org/alpine/v3.16/community/x86_64/APKINDEX.tar.gz
    ERROR: unable to select packages:
      .app-rundeps-20230221.211008:
        masked in: cache
        satisfies: world[.app-rundeps=20230221.211008]
      wkhtmltopdf (no such package):
        required by: .app-rundeps-20230221.211008[wkhtmltopdf]
      ttf-ubuntu-font-family (no such package):
        required by: .app-rundeps-20230221.211008[ttf-ubuntu-font-family]

Los paquetes faltantes son:

    ttf-ubuntu-font-family
    wkhtmltopdf

Así tengo la versión de Ruby y Alpine en el Dockerfile.test

    FROM ruby:2.7.7-alpine3.16

El tema es que para la versión de alpine que me interesa no está disponible el paquete wkhtmltopdf como se puede ver [aquí](https://pkgs.alpinelinux.org/packages?name=wkhtmltopdf&branch=v3.16&repo=&arch=&maintainer=).

Quería subir a Ruby 3.0.x pero parece que ninguno de esos tiene una imagen que sea Alpine versión 14 al menos. La cuál [sí tiene soporte](https://pkgs.alpinelinux.org/packages?name=wkhtmltopdf&branch=v3.14&repo=&arch=&maintainer=) para wkhtmltopdf.

Según esta [respuesta](https://stackoverflow.com/a/59029347/1407371) en Stack Overflow, puedo hacer un `apk add` para instalar directamente el paquete mediante URL:

    apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.10/main/ wkhtmltopdf=0.12.5-r0

O sea que en mi caso para `wkhtmltopdf` debería ser:

    apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.12/community/ wkhtmltopdf=0.12.5-r1

Y para el otro paquete `ttf-ubuntu-font-family`:

    apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.12/main/ ttf-ubuntu-font-family=0.83-r0

Parece que funcionó aunque encontré otro error.

## Error unable to select packages: **so:libicui18n.so.67**
    + apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.12/community/ 'wkhtmltopdf=0.12.5-r1'
    fetch http://dl-cdn.alpinelinux.org/alpine/v3.12/community/x86_64/APKINDEX.tar.gz
    fetch https://dl-cdn.alpinelinux.org/alpine/v3.16/main/x86_64/APKINDEX.tar.gz
    fetch https://dl-cdn.alpinelinux.org/alpine/v3.16/community/x86_64/APKINDEX.tar.gz
    ERROR: unable to select packages:
      so:libicui18n.so.67 (no such package):
      required by: qt5-qtwebkit-5.212.0_alpha4-r11[so:libicui18n.so.67]
      so:libicuuc.so.67 (no such package):
      required by: qt5-qtwebkit-5.212.0_alpha4-r11[so:libicuuc.so.67]

Probé agregando `libicui18n` a la lista de dependencias en el Dockerfile. Pero eso parece no ser el nombre de la librería para Alpine.

Cuando reviso los workflows cuando estaba en Ruby 2.7.1 encuentro estas:

    (6/113) Installing icu-libs (67.1-r0)
    (7/113) Installing icu (67.1-r0)
    (8/113) Installing icu-dev (67.1-r0)

La versión es: 67.1-r0. Esta solo se encuentra en [la versión 3.12](https://pkgs.alpinelinux.org/packages?name=icu-dev&branch=v3.12&repo=&arch=&maintainer=) de Alpine.

Y así las puedo instalar mediante repositorio por URL:

    apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.12/main/ icu-dev=67.1-r0
    apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.12/main/ icu-libs=67.1-r0
    apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.12/main/ icu=67.1-r0


## Primero instalar dependencias y luego software

Cambié el orden de instalación. Moví primero las dependencias arriba y wkhtmltopdf abajo e igual instaló lo que le dio la gana:

    Installing icu-libs (71.1-r2)
    Installing icu (71.1-r2)
    Installing icu-dev (71.1-r2)
    Installing ttf-ubuntu-font-family (0.83-r0)

Instaló las versiones que no eran cuando no había fijado la versión 67.1-r0. Una vez fijadas sí instala pero sale otro error.

## Error conflicto de versiones de paquetes
    ERROR: unable to select packages:
    icu-libs-67.1-r0:
      conflicts: icu-libs-71.1-r2
      satisfies: world[icu-libs=67.1-r0]
                 icu-dev-67.1-r0[icu-libs=67.1-r0]
                 qt5-qtbase-5.14.2-r1[so:libicui18n.so.67]
                 qt5-qtbase-5.14.2-r1[so:libicuuc.so.67]
                 icu-67.1-r0[so:libicui18n.so.67]
                 icu-67.1-r0[so:libicuio.so.67]
                 icu-67.1-r0[so:libicutu.so.67]
                 icu-67.1-r0[so:libicuuc.so.67]
                 qt5-qtwebkit-5.212.0_alpha4-r11[so:libicui18n.so.67]
                 qt5-qtwebkit-5.212.0_alpha4-r11[so:libicuuc.so.67]
                 qt5-qtlocation-5.14.2-r0[so:libicuuc.so.67]
    icu-libs-71.1-r2:
      conflicts: icu-libs-67.1-r0
      breaks: world[icu-libs=67.1-r0]
              icu-dev-67.1-r0[icu-libs=67.1-r0]
      satisfies: nodejs-16.19.1-r0[so:libicui18n.so.71]
                 nodejs-16.19.1-r0[so:libicuuc.so.71]
    .app-rundeps-20230222.032631:
      masked in: cache
      satisfies: world[.app-rundeps=20230222.032631]

Lo que entiendo es que la versión 71.1-r2 es requerida por:

- nodejs-16.19.1-r0[so:libicui18n.so.71]
- nodejs-16.19.1-r0[so:libicuuc.so.71]

En el build con Ruby 2.7.1 la versión de NodeJS instalada es la 12.22

    Installing nodejs (12.22.12-r0)

Decido instalar nodejs fijado a esa versión:

    nodejs=12.22.12-r0

Finalmente la imagen se construye al fijar las versiones disponibles en Alpine 3.12 pero ahora da un error más maluco.

## Error missing qt5-qt libs

Las librerías del error:

    libQt5Widgets
    libQt5Svg
    libQt5Gui
    libQt5Quick
    libQt5WebChannel
    libQt5Sensors
    libQt5Qml

El error (una parte):

    RuntimeError:
    PDF could not be generated!
     Command Error: Error relocating /usr/lib/libQt5Widgets.so.5: _ZN9QFileInfo4statEv: symbol not found
    Error relocating /usr/lib/libQt5Widgets.so.5: _ZN2Qt3hexER11QTextStream: symbol not found
    Error relocating /usr/lib/libQt5Widgets.so.5: _ZNK7QString5splitE5QChar6QFlagsIN2Qt18SplitBehaviorFlagsEENS2_15CaseSensitivityE: symbol not found
    Error relocating /usr/lib/libQt5Widgets.so.5: _ZN2Qt11noforcesignER11QTextStream: symbol not found
    Error relocating /usr/lib/libQt5Widgets.so.5: _ZN7QThread4waitE14QDeadlineTimer: symbol not found
    Error relocating /usr/lib/libQt5Widgets.so.5: _ZN2Qt3decER11QTextStream: symbol not found
    Error relocating /usr/lib/libQt5Widgets.so.5: _ZNK7QString5splitERK7QRegExp6QFlagsIN2Qt18SplitBehaviorFlagsEE: symbol not found
    Error relocating /usr/lib/libQt5Widgets.so.5: _ZN2Qt4endlER11QTextStream: symbol not found
    Error relocating /usr/lib/libQt5Widgets.so.5: _ZN2Qt9forcesignER11QTextStream: symbol not found
    Error relocating /usr/lib/libQt5Widgets.so.5: 

Versión de Qt5-webkit en el build con Ruby 2.7.1/Alpine 3.12:

    Installing qt5-qtwebkit (5.212.0_alpha4-r11)

Versión instalada con Ruby 2.7.7:

    qt5-qtwebkit (5.212.0_alpha4-r11)

Probaré instalando las librerías que se instalaban con Ruby 2.7.1 en Alpine 3.12

    Installing qt5-qtbase (5.14.2-r1)
    Installing qt5-qtbase-x11 (5.14.2-r1)
    Installing qt5-qtsvg (5.14.2-r0)
    Installing qt5-qtdeclarative (5.14.2-r0)
    Installing qt5-qtlocation (5.14.2-r0)
    Installing qt5-qtsensors (5.14.2-r0)
    Installing qt5-qtwebchannel (5.14.2-r0)

Otra vez vuelve el error de conflictos

    ERROR: unable to select packages:
     icu-libs-67.1-r0:
       conflicts: icu-libs-71.1-r2
       satisfies: world[icu-libs=67.1-r0]
                  icu-dev-67.1-r0[icu-libs=67.1-r0]
                  icu-67.1-r0[so:libicui18n.so.67]
                  icu-67.1-r0[so:libicuio.so.67]
                  icu-67.1-r0[so:libicutu.so.67]
                  icu-67.1-r0[so:libicuuc.so.67]
     icu-libs-71.1-r2:
       conflicts: icu-libs-67.1-r0
       breaks: world[icu-libs=67.1-r0]
               icu-dev-67.1-r0[icu-libs=67.1-r0]
       satisfies: qt5-qtbase-5.15.4_git20220511-r2[so:libicui18n.so.71]
                  qt5-qtbase-5.15.4_git20220511-r2[so:libicuuc.so.71]
     qt5-qtbase-5.15.4_git20220511-r2:
       breaks: world[qt5-qtbase=5.14.2-r1]
       satisfies: qt5-qtbase-x11-5.15.4_git20220511-r2[so:libQt5Core.so.5]
                  qt5-qtbase-x11-5.15.4_git20220511-r2[so:libQt5DBus.so.5]
                  qt5-qtbase-x11-5.15.4_git20220511-r2[so:libQt5Network.so.5]
     qt5-qtbase-x11-5.15.4_git20220511-r2:
       breaks: world[qt5-qtbase-x11=5.14.2-r1]

La cosa está dificilísima. Toca probar subiendo a Ruby 3.x

## Enlaces con respecto a Alpine y sus versiones de paquetes
- Moving away from Alpine https://dev.to/asyazwan/moving-away-from-alpine-30n4
    - Aquí mencionan que Alpine no da soporte a versiones muy viejas.
    - Se elimina el paquete
- Alpine repo drops packages. This prevents package version pinning, and makes apk non-deterministic https://gitlab.alpinelinux.org/alpine/abuild/-/issues/9996
    - Y acá explican que es política de Alpine de no almacenar paquetes muy viejos.

