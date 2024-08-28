# Configurando Grimoire

# Error de protobuff-c al configurar PostGIS

Estos son todos los comandos:

    curl -o /tmp/postgis-3.4.0.tar.gz https://download.osgeo.org/postgis/source/postgis-3.4.0.tar.gz
    tar xzf /tmp/postgis-3.4.0.tar.gz -C /tmp
    pushd /tmp/postgis-3.4.0
    eval ./configure `pg_config --configure` --with-projdir=$(brew --cellar proj)/$(pkg-config --modversion proj) --with-pgconfig=$(which pg_config)
    make
    make install
    popd

Me dio este error:

    $ eval ./configure `pg_config --configure` --with-projdir=$(brew --cellar proj)/$(pkg-config --modversion proj) --with-pgconfig=$(which pg_config)
    
    configure: error: unable to find protobuf-c/protobuf-c.h using CPPFLAGS. You can disable MVT and Geobuf support using --without-protobuf

Así que lo hice como sugieren e instaló:

    $ eval ./configure `pg_config --configure` --with-projdir=$(brew --cellar proj)/$(pkg-config --modversion proj) --with-pgconfig=$(which pg_config) --without-protobuf
    PostGIS is now configured for arm-apple-darwin22.3.0


# Error de comando Make al instalar PostGIS
    in /tmp/postgis-3.4.0
    $ make
    
    parseaddress-api.c:25:11: fatal error: 'pcre.h' file not found
    # include <pcre.h>
              ^~~~~~~~
    1 warning and 1 error generated.
    make[2]: *** [parseaddress-api.o] Error 1
    make[1]: *** [all] Error 1
    make: *** [all] Error 1



# Otros Detalles
## Todos los mensajes luego de configurar PostGIS
    PostGIS is now configured for arm-apple-darwin22.3.0
    
     -------------- Compiler Info ------------- 
      C compiler:           clang -std=gnu99 -g -O2 -fno-math-errno -fno-signed-zeros -Wall -g
      C++ compiler (FlatGeobuf): clang -std=c++11 -x c++ 
      CPPFLAGS:              -I/opt/homebrew/Cellar/geos/3.12.1/include -I/opt/homebrew/Cellar/proj/9.3.1/include  -I/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include -I/opt/homebrew/Cellar/sfcgal/1.5.1_1/include  -I/usr/local/include -I/opt/homebrew/opt/openssl@3/include -I/opt/homebrew/opt/readline/include
      LDFLAGS:              -L/opt/homebrew/opt/openssl@3/lib -L/opt/homebrew/opt/readline/lib -lm
      SQL preprocessor:     /usr/bin/cpp -traditional-cpp -w -P -Upixel -Ubool
      Archiver:             ar rs
    
     -------------- Additional Info ------------- 
      Interrupt Tests:   ENABLED
    
     -------------- Dependencies -------------- 
      GEOS config:          /opt/homebrew/bin/geos-config
      GEOS version:         3.12.1
      GDAL config:          /opt/homebrew/bin/gdal-config
      GDAL version:         3.8.3
      SFCGAL config:        /opt/homebrew/bin/sfcgal-config
      SFCGAL version:       1.5.1
      PostgreSQL config:    /opt/homebrew/bin/pg_config
      PostgreSQL version:   PostgreSQL 14.10 (Homebrew)
      PROJ4 version:        93
      Libxml2 config:       /usr/bin/xml2-config
      Libxml2 version:      2.9.13
      JSON-C support:       no
      protobuf support:     no
      PCRE support:         Version 1
      Perl:                 /usr/bin/perl
    
     --------------- Extensions --------------- 
      PostgreSQL EXTENSION support:       enabled
      PostGIS Raster:                     enabled
      PostGIS Topology:                   enabled
      SFCGAL support:                     enabled
      Address Standardizer support:       enabled
    
     -------- Documentation Generation -------- 
      xsltproc:             /usr/bin/xsltproc
      xsl style sheets:     
      dblatex:              
      convert:              /opt/homebrew/bin/convert
      mathml2.dtd:          http://www.w3.org/Math/DTD/mathml2/mathml2.dtd
    
    configure: WARNING: 
    configure: WARNING:  | You are building using --with-projdir. This option isn't standard and    |
    configure: WARNING:  | might be incompatible with future releases of PROJ.                      |
    configure: WARNING:  | You can instead adjust the PKG_CONFIG_PATH environment variable if you   |
    configure: WARNING:  | installed software in a non-standard prefix.                             |
    configure: WARNING:  | Alternatively, you may set the environment variables PROJ_CFLAGS and     |
    configure: WARNING:  | PROJ_LIBS to avoid the need to call pkg-config.                          |

