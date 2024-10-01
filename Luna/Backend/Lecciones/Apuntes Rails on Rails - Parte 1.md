# Apuntes Ruby on Rails - Parte 1

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


# (BUG) Segmentation fault at - Rails Console

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

```bash
gem install eventmachine -- --with-openssl-dir=$(brew --prefix openssl@1.1)
```

## Actualización: 24 Septiembre 2024

El comando:
```bash
gem install eventmachine -- --with-openssl-dir=$(brew --prefix openssl@1.1)
```

funcionó esta vez y ahora sí pude completar la instalación:
```bash
$ gem install mailcatcher --no-document
Successfully installed daemons-1.4.1
Building native extensions. This could take a while...
Successfully installed thin-1.8.2
Successfully installed sinatra-3.2.0
Successfully installed faye-websocket-0.11.3
Successfully installed mailcatcher-0.10.0
5 gems installed
```

# Vistas y HTML

Vi que en el archivo `active_admin_logged_out.html.haml` se usaba el método `render_or_call_method_or_proc_on` así:

```ruby
%title= [@page_title, render_or_call_method_or_proc_on(self, ActiveAdmin.application.site_title)].compact.join(" | ")
```

Llama la atención que hace eso. La documentación está aquí -> https://www.rubydoc.info/gems/activeadmin-rails/MethodOrProcHelper:render_or_call_method_or_proc_on

## Rubocop y Haml-lint

Este proyecto tiene configurado rubocop y haml-lint como linters. Rubocop se encarga de archivos meramente Ruby y haml-lint las vistas escritas en Haml.

Haml-lint puede tomar como [fuente de configuraciones](https://github.com/sds/haml-lint/blob/main/lib/haml_lint/linter/README.md#rubocop) lo que se define en `.rubocop.yml` así que si se necesita excluir algo para que no revise los archivo .haml, hay que cambiar la configuración en el archivo de rubocop.

El comando de haml-lint se corre así:
```bash
bundle exec haml-lint app/views/**/*.haml
```

# Rutas

Al declarar una ruta de esta forma:
```ruby
get "verify_email/:base64_token/failure" => "user_communication_methods/email_verifications#failure"
```

para poder usarla en el controlador, por ejemplo:
```ruby
redirect_to verify_email_failure_path(base64_token: params.require(:base64_token))
```

hay que definir la ruta pasado la opción `as:`, de esta forma:
```ruby
get "verify_email/:base64_token/failure" => "user_communication_methods/email_verifications#failure", as: "verify_email_failure"
```

De lo contrario la ruta no se podrá usar ya que esa sintaxis de ruta por string no genera los helpers.

# Matar servidor Rails en puerto 3000

Así:
```bash
kill -9 $(lsof -i tcp:3000 -t)
```

# Asociación con scope usando enum

Vi esto en el concern `PortalProviderEntity`:
```ruby
has_many :portal_email_recipients,
	 -> { portal_email_recipient }, as: :parent, class_name: "ShadowUser"
```

y esto en el modelo `Physician`:
```ruby
has_many :escalation_email_recipients,
    -> { escalation_email_recipient }, as: :parent, class_name: "ShadowUser"
```

No entendía que es el valor referenciado en la lambda. De dónde salía. La conclusión es que eso es el valor del enum que está definido en el modelo `ShadowUser`:
```ruby
enum kind: {
	portal_email_recipient: 0,
	escalation_email_recipient: 1
}
```