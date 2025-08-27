# Configurando Grimoire

## Error de protobuff-c al configurar PostGIS

Estos son todos los comandos:
```
curl -o /tmp/postgis-3.4.0.tar.gz https://download.osgeo.org/postgis/source/postgis-3.4.0.tar.gz
tar xzf /tmp/postgis-3.4.0.tar.gz -C /tmp
pushd /tmp/postgis-3.4.0
eval ./configure `pg_config --configure` --with-projdir=$(brew --cellar proj)/$(pkg-config --modversion proj) --with-pgconfig=$(which pg_config)
make
make install
popd
```

Me dio este error:
```
$ eval ./configure `pg_config --configure` --with-projdir=$(brew --cellar proj)/$(pkg-config --modversion proj) --with-pgconfig=$(which pg_config)

configure: error: unable to find protobuf-c/protobuf-c.h using CPPFLAGS. You can disable MVT and Geobuf support using --without-protobuf
```

Así que lo hice como sugieren e instaló:
```
$ eval ./configure `pg_config --configure` --with-projdir=$(brew --cellar proj)/$(pkg-config --modversion proj) --with-pgconfig=$(which pg_config) --without-protobuf
PostGIS is now configured for arm-apple-darwin22.3.0
```


## Error de comando Make al instalar PostGIS

```
in /tmp/postgis-3.4.0
$ make

parseaddress-api.c:25:11: fatal error: 'pcre.h' file not found
# include <pcre.h>
				^~~~~~~~
1 warning and 1 error generated.
make[2]: *** [parseaddress-api.o] Error 1
make[1]: *** [all] Error 1
make: *** [all] Error 1
```


# Mensajes de Confirmación cuando todo sale bien

## PostGIS

### Julio 11, 2025

Después del comando:
```
eval ./configure `pg_config --configure` --with-projdir=$(brew --cellar proj)/$(pkg-config --modversion proj) --with-pgconfig=$(which pg_config) --without-protobuf
```

Esto:
```
  PostGIS is now configured for arm-apple-darwin22.3.0

 -------------- Compiler Info -------------
  C compiler:           clang -std=gnu99 -g -O2 -fno-math-errno -fno-signed-zeros -Wall -g
  C++ compiler (FlatGeobuf): clang -std=c++11 -x c++
  CPPFLAGS:              -I/opt/homebrew/Cellar/geos/3.13.0/include -I/opt/homebrew/Cellar/proj/9.5.1/include  -I/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include -I/opt/homebrew/Cellar/sfcgal/2.0.0_1/include -I/opt/homebrew/Cellar/json-c/0.18/include -I/opt/homebrew/Cellar/json-c/0.18/include/json-c -I/opt/homebrew/Cellar/pcre2/10.44/include -I/opt/homebrew/opt/gettext/include -I/opt/homebrew/opt/krb5/include -I/opt/homebrew/opt/openssl@3/include -I/opt/homebrew/opt/readline/include
  LDFLAGS:              -L/opt/homebrew/opt/gettext/lib -L/opt/homebrew/opt/krb5/lib -L/opt/homebrew/opt/openssl@3/lib -L/opt/homebrew/opt/readline/lib -lm
  SQL preprocessor:     /usr/bin/cpp -traditional-cpp -w -P -Upixel -Ubool
  Archiver:             ar rs

 -------------- Additional Info -------------
  Interrupt Tests:   ENABLED

 -------------- Dependencies --------------
  GEOS config:          /opt/homebrew/bin/geos-config
  GEOS version:         3.13.0
  GDAL config:          /opt/homebrew/bin/gdal-config
  GDAL version:         3.10.1
  SFCGAL config:        /opt/homebrew/bin/sfcgal-config
  SFCGAL version:       2.0.0
  PostgreSQL config:    /opt/homebrew/opt/postgresql@17/bin/pg_config
  PostgreSQL version:   PostgreSQL 17.3 (Homebrew)
  PROJ4 version:        95
  Libxml2 config:       /usr/bin/xml2-config
  Libxml2 version:      2.9.13
  JSON-C support:       yes
  protobuf support:     no
  PCRE support:         Version 2
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
  convert:              /opt/homebrew/opt/imagemagick@6/bin/convert
  mathml2.dtd:          http://www.w3.org/Math/DTD/mathml2/mathml2.dtd

configure: WARNING:
configure: WARNING:  | You are building using --with-projdir. This option isn't standard and    |
configure: WARNING:  | might be incompatible with future releases of PROJ.                      |
configure: WARNING:  | You can instead adjust the PKG_CONFIG_PATH environment variable if you   |
configure: WARNING:  | installed software in a non-standard prefix.                             |
configure: WARNING:  | Alternatively, you may set the environment variables PROJ_CFLAGS and     |
configure: WARNING:  | PROJ_LIBS to avoid the need to call pkg-config.
```

Después del comando:
```
make
```

Esto:

```
rm -f sql/test-debug_standardize_address.sql.tmp
echo '-- Just tag extension address_standardizer version as "ANY"' > sql/address_standardizer--TEMPLATED--TO--ANY.sql
echo '-- Installed by address_standardizer 3.4.2' >> sql/address_standardizer--TEMPLATED--TO--ANY.sql
echo '-- Built on Fri Jul 11 13:57:54 -05 2025' >> sql/address_standardizer--TEMPLATED--TO--ANY.sql
cat address_standardizer.control.in \
                | /usr/bin/perl -lpe "s'@EXTVERSION@'3.4.2'g" \
                > address_standardizer.control
cat address_standardizer_data_us.control.in \
                | /usr/bin/perl -lpe "s'@EXTVERSION@'3.4.2'g" \
                > address_standardizer_data_us.control
PostGIS was built successfully. Ready to install.
```

Después de `make install`:
```
/Library/Developer/CommandLineTools/usr/bin/make -C extensions/ install-extension-upgrades-from-known-versions
for DIR in postgis postgis_tiger_geocoder postgis_raster postgis_topology postgis_sfcgal address_standardizer; do \
                echo "---- Making install-extension-upgrades-from-known-versions in ${DIR}"; \
                /Library/Developer/CommandLineTools/usr/bin/make -C "${DIR}" install-extension-upgrades-from-known-versions || exit 1; \
        done
---- Making install-extension-upgrades-from-known-versions in postgis
/usr/bin/perl ../../loader/postgis.pl \
                install-extension-upgrades \
                --extension postgis \
                --pg_sharedir /opt/homebrew/share/postgresql@17 \
                2.0.0 2.0.1 2.0.2 2.0.3 2.0.4 2.0.5 2.0.6 2.0.7 2.1.0 2.1.1 2.1.2 2.1.3 2.1.4 2.1.5 2.1.6 2.1.7 2.1.8 2.1.9 2.2.0 2.2.1 2.2.2 2.2.3 2.2.4 2.2.5 2.2.6 2.2.7 2.2.8 2.3.0 2.3.1 2.3.2 2.3.3 2.3.4 2.3.5 2.3.6 2.3.7 2.3.8 2.3.9 2.3.10 2.3.11 2.4.0 2.4.1 2.4.2 2.4.3 2.4.4 2.4.5 2.4.6 2.4.7 2.4.8 2.4.9 2.4.10 2.5.0 2.5.1 2.5.2 2.5.3 2.5.4 2.5.5 2.5.6 2.5.7 2.5.8 2.5.9 3.0.0 3.0.1 3.0.2 3.0.3 3.0.4 3.0.5 3.0.6 3.0.7 3.0.8 3.0.9 3.0.10 3.1.0 3.1.1 3.1.2 3.1.3 3.1.4 3.1.5 3.1.6 3.1.7 3.1.8 3.1.9 3.1.10 3.2.0 3.2.1 3.2.2 3.2.3 3.2.4 3.2.5 3.2.6 3.3.0 3.3.1 3.3.2 3.3.3 3.3.4 3.3.5 3.4.0 3.4.1
Installation target: /opt/homebrew/share/postgresql@17 postgis
---- Making install-extension-upgrades-from-known-versions in postgis_tiger_geocoder
/usr/bin/perl ../../loader/postgis.pl \
                install-extension-upgrades \
                --extension postgis_tiger_geocoder \
                --pg_sharedir /opt/homebrew/share/postgresql@17 \
                2.0.0 2.0.1 2.0.2 2.0.3 2.0.4 2.0.5 2.0.6 2.0.7 2.1.0 2.1.1 2.1.2 2.1.3 2.1.4 2.1.5 2.1.6 2.1.7 2.1.8 2.1.9 2.2.0 2.2.1 2.2.2 2.2.3 2.2.4 2.2.5 2.2.6 2.2.7 2.2.8 2.3.0 2.3.1 2.3.2 2.3.3 2.3.4 2.3.5 2.3.6 2.3.7 2.3.8 2.3.9 2.3.10 2.3.11 2.4.0 2.4.1 2.4.2 2.4.3 2.4.4 2.4.5 2.4.6 2.4.7 2.4.8 2.4.9 2.4.10 2.5.0 2.5.1 2.5.2 2.5.3 2.5.4 2.5.5 2.5.6 2.5.7 2.5.8 2.5.9 3.0.0 3.0.1 3.0.2 3.0.3 3.0.4 3.0.5 3.0.6 3.0.7 3.0.8 3.0.9 3.0.10 3.1.0 3.1.1 3.1.2 3.1.3 3.1.4 3.1.5 3.1.6 3.1.7 3.1.8 3.1.9 3.1.10 3.2.0 3.2.1 3.2.2 3.2.3 3.2.4 3.2.5 3.2.6 3.3.0 3.3.1 3.3.2 3.3.3 3.3.4 3.3.5 3.4.0 3.4.1
Installation target: /opt/homebrew/share/postgresql@17 postgis_tiger_geocoder
---- Making install-extension-upgrades-from-known-versions in postgis_raster
/usr/bin/perl ../../loader/postgis.pl \
                install-extension-upgrades \
                --extension postgis_raster \
                --pg_sharedir /opt/homebrew/share/postgresql@17 \
                2.0.0 2.0.1 2.0.2 2.0.3 2.0.4 2.0.5 2.0.6 2.0.7 2.1.0 2.1.1 2.1.2 2.1.3 2.1.4 2.1.5 2.1.6 2.1.7 2.1.8 2.1.9 2.2.0 2.2.1 2.2.2 2.2.3 2.2.4 2.2.5 2.2.6 2.2.7 2.2.8 2.3.0 2.3.1 2.3.2 2.3.3 2.3.4 2.3.5 2.3.6 2.3.7 2.3.8 2.3.9 2.3.10 2.3.11 2.4.0 2.4.1 2.4.2 2.4.3 2.4.4 2.4.5 2.4.6 2.4.7 2.4.8 2.4.9 2.4.10 2.5.0 2.5.1 2.5.2 2.5.3 2.5.4 2.5.5 2.5.6 2.5.7 2.5.8 2.5.9 3.0.0 3.0.1 3.0.2 3.0.3 3.0.4 3.0.5 3.0.6 3.0.7 3.0.8 3.0.9 3.0.10 3.1.0 3.1.1 3.1.2 3.1.3 3.1.4 3.1.5 3.1.6 3.1.7 3.1.8 3.1.9 3.1.10 3.2.0 3.2.1 3.2.2 3.2.3 3.2.4 3.2.5 3.2.6 3.3.0 3.3.1 3.3.2 3.3.3 3.3.4 3.3.5 3.4.0 3.4.1
Installation target: /opt/homebrew/share/postgresql@17 postgis_raster
---- Making install-extension-upgrades-from-known-versions in postgis_topology
/usr/bin/perl ../../loader/postgis.pl \
                install-extension-upgrades \
                --extension postgis_topology \
                --pg_sharedir /opt/homebrew/share/postgresql@17 \
                2.0.0 2.0.1 2.0.2 2.0.3 2.0.4 2.0.5 2.0.6 2.0.7 2.1.0 2.1.1 2.1.2 2.1.3 2.1.4 2.1.5 2.1.6 2.1.7 2.1.8 2.1.9 2.2.0 2.2.1 2.2.2 2.2.3 2.2.4 2.2.5 2.2.6 2.2.7 2.2.8 2.3.0 2.3.1 2.3.2 2.3.3 2.3.4 2.3.5 2.3.6 2.3.7 2.3.8 2.3.9 2.3.10 2.3.11 2.4.0 2.4.1 2.4.2 2.4.3 2.4.4 2.4.5 2.4.6 2.4.7 2.4.8 2.4.9 2.4.10 2.5.0 2.5.1 2.5.2 2.5.3 2.5.4 2.5.5 2.5.6 2.5.7 2.5.8 2.5.9 3.0.0 3.0.1 3.0.2 3.0.3 3.0.4 3.0.5 3.0.6 3.0.7 3.0.8 3.0.9 3.0.10 3.1.0 3.1.1 3.1.2 3.1.3 3.1.4 3.1.5 3.1.6 3.1.7 3.1.8 3.1.9 3.1.10 3.2.0 3.2.1 3.2.2 3.2.3 3.2.4 3.2.5 3.2.6 3.3.0 3.3.1 3.3.2 3.3.3 3.3.4 3.3.5 3.4.0 3.4.1
Installation target: /opt/homebrew/share/postgresql@17 postgis_topology
---- Making install-extension-upgrades-from-known-versions in postgis_sfcgal
/usr/bin/perl ../../loader/postgis.pl \
                install-extension-upgrades \
                --extension postgis_sfcgal \
                --pg_sharedir /opt/homebrew/share/postgresql@17 \
                2.0.0 2.0.1 2.0.2 2.0.3 2.0.4 2.0.5 2.0.6 2.0.7 2.1.0 2.1.1 2.1.2 2.1.3 2.1.4 2.1.5 2.1.6 2.1.7 2.1.8 2.1.9 2.2.0 2.2.1 2.2.2 2.2.3 2.2.4 2.2.5 2.2.6 2.2.7 2.2.8 2.3.0 2.3.1 2.3.2 2.3.3 2.3.4 2.3.5 2.3.6 2.3.7 2.3.8 2.3.9 2.3.10 2.3.11 2.4.0 2.4.1 2.4.2 2.4.3 2.4.4 2.4.5 2.4.6 2.4.7 2.4.8 2.4.9 2.4.10 2.5.0 2.5.1 2.5.2 2.5.3 2.5.4 2.5.5 2.5.6 2.5.7 2.5.8 2.5.9 3.0.0 3.0.1 3.0.2 3.0.3 3.0.4 3.0.5 3.0.6 3.0.7 3.0.8 3.0.9 3.0.10 3.1.0 3.1.1 3.1.2 3.1.3 3.1.4 3.1.5 3.1.6 3.1.7 3.1.8 3.1.9 3.1.10 3.2.0 3.2.1 3.2.2 3.2.3 3.2.4 3.2.5 3.2.6 3.3.0 3.3.1 3.3.2 3.3.3 3.3.4 3.3.5 3.4.0 3.4.1
Installation target: /opt/homebrew/share/postgresql@17 postgis_sfcgal
---- Making install-extension-upgrades-from-known-versions in address_standardizer
Makefile:230: warning: overriding commands for target `install'
/opt/homebrew/lib/postgresql@17/pgxs/src/makefiles/pgxs.mk:239: warning: ignoring old commands for target `install'
TARGET=`echo install-extension-upgrades-from-known-versions-data | sed 's/-data$//'`; \
        for lang in us; do \
                EXTENSION_DATA_INSTALL=1 \
                /Library/Developer/CommandLineTools/usr/bin/make ${TARGET} \
                EXTENSION=address_standardizer_data_${lang} \
                || exit 1; \
        done
Makefile:230: warning: overriding commands for target `install'
/opt/homebrew/lib/postgresql@17/pgxs/src/makefiles/pgxs.mk:239: warning: ignoring old commands for target `install'
/usr/bin/perl ../../loader/postgis.pl \
                install-extension-upgrades \
                --extension address_standardizer_data_us \
                --pg_sharedir /opt/homebrew/share/postgresql@17 \
                2.0.0 2.0.1 2.0.2 2.0.3 2.0.4 2.0.5 2.0.6 2.0.7 2.1.0 2.1.1 2.1.2 2.1.3 2.1.4 2.1.5 2.1.6 2.1.7 2.1.8 2.1.9 2.2.0 2.2.1 2.2.2 2.2.3 2.2.4 2.2.5 2.2.6 2.2.7 2.2.8 2.3.0 2.3.1 2.3.2 2.3.3 2.3.4 2.3.5 2.3.6 2.3.7 2.3.8 2.3.9 2.3.10 2.3.11 2.4.0 2.4.1 2.4.2 2.4.3 2.4.4 2.4.5 2.4.6 2.4.7 2.4.8 2.4.9 2.4.10 2.5.0 2.5.1 2.5.2 2.5.3 2.5.4 2.5.5 2.5.6 2.5.7 2.5.8 2.5.9 3.0.0 3.0.1 3.0.2 3.0.3 3.0.4 3.0.5 3.0.6 3.0.7 3.0.8 3.0.9 3.0.10 3.1.0 3.1.1 3.1.2 3.1.3 3.1.4 3.1.5 3.1.6 3.1.7 3.1.8 3.1.9 3.1.10 3.2.0 3.2.1 3.2.2 3.2.3 3.2.4 3.2.5 3.2.6 3.3.0 3.3.1 3.3.2 3.3.3 3.3.4 3.3.5 3.4.0 3.4.1
Installation target: /opt/homebrew/share/postgresql@17 address_standardizer_data_us
/usr/bin/perl ../../loader/postgis.pl \
                install-extension-upgrades \
                --extension address_standardizer \
                --pg_sharedir /opt/homebrew/share/postgresql@17 \
                2.0.0 2.0.1 2.0.2 2.0.3 2.0.4 2.0.5 2.0.6 2.0.7 2.1.0 2.1.1 2.1.2 2.1.3 2.1.4 2.1.5 2.1.6 2.1.7 2.1.8 2.1.9 2.2.0 2.2.1 2.2.2 2.2.3 2.2.4 2.2.5 2.2.6 2.2.7 2.2.8 2.3.0 2.3.1 2.3.2 2.3.3 2.3.4 2.3.5 2.3.6 2.3.7 2.3.8 2.3.9 2.3.10 2.3.11 2.4.0 2.4.1 2.4.2 2.4.3 2.4.4 2.4.5 2.4.6 2.4.7 2.4.8 2.4.9 2.4.10 2.5.0 2.5.1 2.5.2 2.5.3 2.5.4 2.5.5 2.5.6 2.5.7 2.5.8 2.5.9 3.0.0 3.0.1 3.0.2 3.0.3 3.0.4 3.0.5 3.0.6 3.0.7 3.0.8 3.0.9 3.0.10 3.1.0 3.1.1 3.1.2 3.1.3 3.1.4 3.1.5 3.1.6 3.1.7 3.1.8 3.1.9 3.1.10 3.2.0 3.2.1 3.2.2 3.2.3 3.2.4 3.2.5 3.2.6 3.3.0 3.3.1 3.3.2 3.3.3 3.3.4 3.3.5 3.4.0 3.4.1
Installation target: /opt/homebrew/share/postgresql@17 address_standardizer
```

## Vector

Comando `make`:
```
clang -Wall -Wmissing-prototypes -Wpointer-arith -Wdeclaration-after-statement -Werror=vla -Werror=unguarded-availability-new -Wendif-labels -Wmissing-format-attribute -Wcast-function-type -Wformat-security -fno-strict-aliasing -fwrapv -Wno-unused-command-line-argument -Wno-compound-token-split-by-macro -O2  -ftree-vectorize -fassociative-math -fno-signed-zeros -fno-trapping-math  -fvisibility=hidden -bundle -o vector.dylib src/hnsw.o src/hnswbuild.o src/hnswinsert.o src/hnswscan.o src/hnswutils.o src/hnswvacuum.o src/ivfbuild.o src/ivfflat.o src/ivfinsert.o src/ivfkmeans.o src/ivfscan.o src/ivfutils.o src/ivfvacuum.o src/vector.o -L/opt/homebrew/lib/postgresql@17  -isysroot /Library/Developer/CommandLineTools/SDKs/MacOSX13.sdk -L/opt/homebrew/opt/gettext/lib -L/opt/homebrew/opt/krb5/lib -L/opt/homebrew/opt/openssl@3/lib -L/opt/homebrew/opt/readline/lib -L/opt/homebrew/opt/lz4/lib -L/opt/homebrew/opt/zstd/lib  -Wl,-dead_strip_dylibs   -fvisibility=hidden -bundle_loader /opt/homebrew/Cellar/postgresql@17/17.3/bin/postgres
cp sql/vector.sql sql/vector--0.5.0.sql
```

Comando `make install`:
```
/usr/bin/install -c -m 755  vector.dylib '/opt/homebrew/lib/postgresql@17/vector.dylib'
/usr/bin/install -c -m 644 .//vector.control '/opt/homebrew/share/postgresql@17/extension/'
/usr/bin/install -c -m 644 .//sql/vector--0.1.0--0.1.1.sql .//sql/vector--0.1.1--0.1.3.sql .//sql/vector--0.1.3--0.1.4.sql .//sql/vector--0.1.4--0.1.5.sql .//sql/vector--0.1.5--0.1.6.sql .//sql/vector--0.1.6--0.1.7.sql .//sql/vector--0.1.7--0.1.8.sql .//sql/vector--0.1.8--0.2.0.sql .//sql/vector--0.2.0--0.2.1.sql .//sql/vector--0.2.1--0.2.2.sql .//sql/vector--0.2.2--0.2.3.sql .//sql/vector--0.2.3--0.2.4.sql .//sql/vector--0.2.4--0.2.5.sql .//sql/vector--0.2.5--0.2.6.sql .//sql/vector--0.2.6--0.2.7.sql .//sql/vector--0.2.7--0.3.0.sql .//sql/vector--0.3.0--0.3.1.sql .//sql/vector--0.3.1--0.3.2.sql .//sql/vector--0.3.2--0.4.0.sql .//sql/vector--0.4.0--0.4.1.sql .//sql/vector--0.4.1--0.4.2.sql .//sql/vector--0.4.2--0.4.3.sql .//sql/vector--0.4.3--0.4.4.sql .//sql/vector--0.4.4--0.5.0.sql .//sql/vector--0.5.0.sql  '/opt/homebrew/share/postgresql@17/extension/'
/bin/sh /opt/homebrew/lib/postgresql@17/pgxs/src/makefiles/../../config/install-sh -c -d '/opt/homebrew/include/postgresql@17/server/extension/vector/'
/usr/bin/install -c -m 644   .//src/vector.h '/opt/homebrew/include/postgresql@17/server/extension/vector/'
```