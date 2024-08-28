# Apuntes Rails - Edge #1

# Error al instalar puma 6.3.1

Falló con este error:
```bash
current directory: /Users/francisco/.gem/ruby/3.0.6/gems/puma-6.3.1/ext/puma_http11
make DESTDIR\=
compiling http11_parser.c
compiling mini_ssl.c
compiling puma_http11.c
linking shared-object puma/puma_http11.bundle
Undefined symbols for architecture arm64:
  "_SSL_get1_peer_certificate", referenced from:
	  _engine_peercert in mini_ssl.o
ld: symbol(s) not found for architecture arm64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [puma_http11.bundle] Error 1

make failed, exit code 2
```

Pude instalar Puma de esta forma:
```bash
$ PUMA_DISABLE_SSL=1 gem install puma -v "6.3.1"
Building native extensions. This could take a while...
Successfully installed puma-6.3.1
Parsing documentation for puma-6.3.1
Installing ri documentation for puma-6.3.1
Done installing documentation for puma after 0 seconds
1 gem installed
```

Visto en este [comentario](https://github.com/puma/puma/issues/2328#issuecomment-1673028216).


# Puma compiled without SSL support (RuntimeError)

*Esto para Mac con chip M1.*

Lo de arriba sirve pero al lanzar el servidor Rails en el proyecto backend se quejó con ese error

La solución la encontré en este [comentario](https://github.com/puma/puma/issues/2790#issuecomment-1547332463):

```bash
bundle config build.puma --with-pkg-config=$(brew --prefix openssl@1.1)/lib/pkgconfig
bundle install --redownload
```


# [BUG] Segmentation fault at - Rails Console

Cuando intento hacer cualquier operación en la consola de rails, obtengo este error bien hp:
```ruby
[1] pry(main)> Workout.find("40218ba7-b5fd-479d-a551-5354cb9721b3")
/Users/francisco/.gem/ruby/3.0.6/gems/pg-1.2.3/lib/pg.rb:58: [BUG] Segmentation fault at 0x0000000000000110
ruby 3.0.6p216 (2023-03-30 revision 23a532679b) [arm64-darwin22]

-- Crash Report log information --------------------------------------------
   See Crash Report log file under the one of following:                    
	 * ~/Library/Logs/DiagnosticReports                                     
	 * /Library/Logs/DiagnosticReports                                      
   for more details.                                                        
Don't forget to include the above Crash Report log file in bug reports.     

-- Control frame information -----------------------------------------------
c:0073 p:---- s:0413 e:000412 CFUNC  :initialize
c:0072 p:---- s:0410 e:000409 CFUNC  :new
c:0071 p:0019 s:0405 e:000404 METHOD /Users/francisco/.gem/ruby/3.0.6/gems/pg-1.2.3/lib/pg.rb:58
```

Encontré en esta [respuesta](https://github.com/ged/ruby-pg/issues/538#issuecomment-1591629049) que se debe setear esta variable:
```bash
export PGGSSENCMODE="disable"
```

Solo me pasó en el proyecto Backend/Edge.

# Error al intentar instalar mailcatcher

```bash
1 warning generated.
compiling ssl.cpp
linking shared-object rubyeventmachine.bundle
Undefined symbols for architecture arm64:
  "_SSL_get1_peer_certificate", referenced from:
	  SslBox_t::GetPeerCert() in ssl.o
ld: symbol(s) not found for architecture arm64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [rubyeventmachine.bundle] Error 1

make failed, exit code 2

Gem files will remain installed in /Users/francisco/.gem/ruby/3.0.4/gems/eventmachine-1.0.9 for inspection.
Results logged to /Users/francisco/.gem/ruby/3.0.4/extensions/arm64-darwin-22/3.0.0/eventmachine-1.0.9/gem_make.out
```

Parece que el tema es con la gema Event Machine.

En este [issue](https://github.com/sj26/mailcatcher/issues/254) mencionan usar una ENV toda rara pero no funciona.

```bash
PKG_CONFIG_PATH="$(brew --prefix openssl)/lib/pkgconfig" gem install eventmachine -v "1.0.9"
```

Encontré en el [issue de eventmachine](https://github.com/eventmachine/eventmachine/issues/981) esta alternativa pero no funcionó:

```
gem install eventmachine -- --with-openssl-dir=$(brew --prefix openssl@1.1)
```
