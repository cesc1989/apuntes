# Apuntes Ruby on Rails - Parte 2

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