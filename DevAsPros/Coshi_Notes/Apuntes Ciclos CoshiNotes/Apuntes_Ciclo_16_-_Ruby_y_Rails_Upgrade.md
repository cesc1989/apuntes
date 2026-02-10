# Apuntes Ciclo 16: Upgrades de Ruby y de Rails

# Error de openssl

Si intento instalar ruby 3.2.0 o 3.2.5:
```
==> Upgrading 1 outdated package:
openssl@1.1 1.1.1u -> 1.1.1w
Error: openssl@1.1 has been disabled because it is not supported upstream! It was disabled on 2024-10-24.
!!! Installing dependencies failed!
```

Y aquí el vale de Ruby on Mac ya tiene artículo -> https://www.rubyonmac.dev/openssl-1-1-has-been-disabled

El problema es que ruby-install no está diferenciando la versión de OpenSSL para usar. OpenSSL 1.1 ya llegó al fin de su ciclo de vida y al opción a usar en instalaciones de Ruby superiores a 3.1. De resto, tienen que usar OpenSSL 3.

[Acá hay una tabla de versiones](https://www.rubyonmac.dev/openssl-versions-supported-by-ruby) de Ruby vs OpenSSL.

`ruby-install` ya tiene un fix para [este problema](https://github.com/postmodern/ruby-install/issues/474). En la versión [0.9.3](https://github.com/postmodern/ruby-install/releases/tag/v0.9.3).

Finalmente, la solución a este lío fue actualizar `ruby-install` con brew:
```
brew upgrade ruby-install
```

# ¿Cómo listar paquetes instalados con Homebrew?

Con el comando `brew list` o `brew list --cask`.

```bash
brew list

==> Formulae
abseil			coreutils		giflib			imagemagick@6		libde265		librttopo		libyaml			numpy			pkg-config		readline		utf8proc
aom			eigen			git			imath			libevent		libspatialite		little-cms2		nvm			poppler			redis			webp
apache-arrow		epsilon			glib			isl			libffi			libtasn1		llvm			oniguruma		popt			ruby-build		x265
autoconf		expat			glog			jbig2dec		libgcrypt		libtiff			llvm@18			openblas		postgis			ruby-install		xerces-c
automake		fontconfig		gmp			jpeg-turbo		libgeotiff		libtool			lz4			openexr			postgresql@16		sfcgal			xorgproto
aws-sdk-cpp		freetype		gnupg			jpeg-xl			libgpg-error		libunistring		lzo			openjpeg		proj			shared-mime-info	xz
bison			freexl			gnutls			jq			libheif			libusb			m4			openssl@1.1		protobuf		snappy			z3
boost			fribidi			gobject-introspection	json-c			libidn			libvmaf			minizip			openssl@3		protobuf-c		sqlite			zellij
brotli			fzf			gpgme			krb5			libidn2			libx11			mpdecimal		p11-kit			pyenv			tesseract		zlib
bzip2			gcc			graphite2		kubectx			libkml			libxau			mpfr			packer			python-packaging	the_silver_searcher	zsh
c-ares			gdal			grpc			kubernetes-cli		libksba			libxcb			ncurses			pandoc			python@3.11		thrift			zstd
ca-certificates		gdbm			harfbuzz		leptonica		liblerc			libxdmcp		netcdf			pango			python@3.12		tmux
cairo			geos			hdf5			libaec			libmpc			libxext			nettle			pcre			qhull			tree
cfitsio			gettext			highway			libarchive		libnghttp2		libxml2			npth			pcre2			rapidjson		unbound
cgal			gflags			hub			libassuan		libpng			libxrender		nspr			pinentry		rbenv			unixodbc
chruby			ghostscript		icu4c			libb2			libpq			libxslt			nss			pixman			re2			uriparser

==> Casks
libreoffice	ngrok		wkhtmltopdf
```

# Instalar ruby-install como root y como usuario

> [!Info]
> La documentación de ruby-install es clara sobre esto:
>
> *Supports installing into `/opt/rubies/` for root and `~/.rubies/` for users by default.*

En Cash Cloud, configuré el script para que instale ruby-install [como root](https://github.com/devaspros/cashflow_cloud/blob/main/scripts/001_install_deps.sh#L46-L50). En ese caso, la versión de Ruby instalada queda en la ubicación `/opt/rubies`:

```bash
which ruby
/opt/rubies/ruby-3.1.0/bin/ruby
```

Sin embargo, la versión que instalé como usuario de sistema (`ubuntu`) quedó instalada en otra ubicación:
```bash
ubuntu@localhost:~$ chruby 3.2.5
ubuntu@localhost:~$ which ruby
/home/ubuntu/.rubies/ruby-3.2.5/bin/ruby
```

Para tener en cuenta.