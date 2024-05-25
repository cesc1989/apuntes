# Instalando Ruby en Linux Mint - Ubuntu 22.04
Para instalar la versión 3.1.0 no tuve problemas. En cambio sí los tuve para instalar la versión 3.0.2 (Clinical Dashboard).

La versión 3.1.0 no tiene problemas porque usa versiones más recientes de OpenSSL:

    $ openssl version
    OpenSSL 3.0.2 15 Mar 2022 (Library: OpenSSL 3.0.2 15 Mar 2022)

Desde Ubuntu 22.04 openssl usa la versión 3.

# Instalando ruby 3.0.2

Tope con este error:

    ssl_pkey_rsa.c: At top level:
    cc1: note: unrecognized command-line option ‘-Wno-self-assign’ may have been intended to silence earlier diagnostics
    cc1: note: unrecognized command-line option ‘-Wno-parentheses-equality’ may have been intended to silence earlier diagnostics
    cc1: note: unrecognized command-line option ‘-Wno-constant-logical-operand’ may have been intended to silence earlier diagnostics
    make[2]: *** [Makefile:328: ossl_pkey_rsa.o] Error 1
    make[2]: se sale del directorio '/home/cesc/src/ruby-3.0.2/ext/openssl'
    make[1]: *** [exts.mk:259: ext/openssl/all] Error 2
    make[1]: se sale del directorio '/home/cesc/src/ruby-3.0.2'
    make: *** [uncommon.mk:300: build-ext] Error 2
    !!! Compiling ruby 3.0.2 failed!

Traté lo de [este artículo](https://blog.timothybryantjr.com/posts/installing-ruby-3-0-0-on-ubuntu-22-04-using-rbenv/) pero no funcionó.

    sudo apt install build-essential checkinstall zlib1g-dev
    
    wget https://www.openssl.org/source/old/1.1.1/openssl-1.1.1p.tar.gz
    
    tar -xvf openssl-1.1.1p.tar.gz
    
    ./config --prefix=/opt/openssl-1.1.1q --openssldir=/opt/openssl-1.1.1q shared zlib
    
    make
    make test
    sudo make install
    
    sudo rm -rf /opt/openssl-1.1.1q/certs
    sudo ln -s /etc/ssl/certs /opt/openssl-1.1.1q
    
    # Esta no funco. Parece que es de rbenv
    RUBY_CONFIGURE_OPTS=--with-openssl-dir=/opt/openssl-1.1.1q rbenv install 3.0.0
    
    # Esta tampoco funcó.
    ruby-install 3.0.2 -- --with-openssl-dir=/opt/openssl-1.1.1q

Este [otro artículo sugiere](https://deanpcmad.com/2024/installing-older-ruby-versions-on-ubuntu-24-04-and-22-04/) lo mismo que el anterior.

--

En este [issue en el bug tracker de Ruby](https://bugs.ruby-lang.org/issues/18658) se comenta que en la versión 22.04 de Ubuntu ya no habría soporte para las versiones 3.0.x de Ruby. Justo el problema que estoy teniendo.

Richard Schneeman:

> Ubuntu 22.04 is being released soon and ships with **openssl 3**. As of now Ruby 2.7.x and 3.0.x are under core support and will not build on Ubuntu 22.04 with openssl (Ruby 3.1.x can compile).
> 
> When attempting to compile 3.0.3 on Ubuntu 22 it issues this warning:
    *** Following extensions are not compiled:
    openssl:
      Could not be configured. It will not be installed.
      /ruby-3.0.3/ext/openssl/extconf.rb:113: OpenSSL >= 1.0.1, < 3.0.0 or LibreSSL >= 2.5.0 is required
      Check ext/openssl/mkmf.log for more details.


## Sin openssl

Intenté este comando pero no funcionó:

    RUBY_CONFIGURE_OPTS="--without-openssl" ruby-install 3.0.2

Lo saqué de [este gist](https://gist.github.com/yob/08d53a003181aa0fcce9812b1b533870) (mencionado en el issue tracker oficial). El resumen es que se intenta instalar ruby 3.0.2 sin openssl y luego se instala la gema de openssl de manera manual.

Ahora intenté este otro comando:

    ruby-install 3.0.2 -- --without-openssl


## Bison 2 y openssl 1 - Esta fue la forma correcta

[Este artículo sugiere](https://abevoelker.com/building-old-rubies-on-ubuntu-22.04/) que los errores que se ven son por versiones no soportadas de Bison.

Ubuntu 22.04 viene con Bison 3:

    bison -V
    bison (GNU Bison) 3.8.2

Así se puede tener otra versión de Bison para poder instalar Ruby 3.0.2

    cd
    wget http://ftp.gnu.org/gnu/bison/bison-2.7.1.tar.gz
    tar -xvf bison-2.7.1.tar.gz
    cd bison-2.7.1
    ./configure --prefix=/tmp
    make

pero eso dará error:

    fseterr.c:77:3: error: #error "Please port gnulib fseterr.c to your platform! Look at the definitions of ferror and clearerr on your system, then report this to bug-gnulib."
       77 |  #error "Please port gnulib fseterr.c to your platform! Look at the definitions of ferror and clearerr on your system, then report this to bug-gnulib."
          |   ^~~~~
    make[3]: *** [Makefile:1915: fseterr.o] Error 1

así que toca también:

    wget https://raw.githubusercontent.com/rdslw/openwrt/e5d47f32131849a69a9267de51a30d6be1f0d0ac/tools/bison/patches/110-glibc-change-work-around.patch
    patch -p1 < 110-glibc-change-work-around.patch

Se completa así la instalación de Bison 2:

    make
    make install

Y así se usaría Bison 2:

    PATH=/tmp/bin:$PATH bison -V
    bison (GNU Bison) 2.7.12-4996

Y así volví a intentar pero me faltó lo de OpenSSL v1

    PATH=/tmp/bin:$PATH ruby-install ruby 3.0.2

Debería ser así:

    PATH=/tmp/bin:$PATH ruby-install ruby 3.0.2 -- --with-openssl-dir=/opt/openssl-1.1.1q

Nota que hace la misma instalación de openssl v1 como en el artículo de más arriba.

# Referencias

Todos los artículos mencionados:

- Building old Rubies on Ubuntu 22.04 → https://abevoelker.com/building-old-rubies-on-ubuntu-22.04/ ⭐ 
- Installing ruby-3.0.0 on Ubuntu 22.04 using rbenv → https://blog.timothybryantjr.com/posts/installing-ruby-3-0-0-on-ubuntu-22-04-using-rbenv/
- Installing Older Ruby Versions on Ubuntu 24.04 and 22.04 → https://deanpcmad.com/2024/installing-older-ruby-versions-on-ubuntu-24-04-and-22-04/



