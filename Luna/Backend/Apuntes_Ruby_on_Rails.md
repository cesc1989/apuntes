# Apuntes Ruby on Rails - Parte 1

# Vistas y HTML

Vi que en el archivo `active_admin_logged_out.html.haml` se usaba el método `render_or_call_method_or_proc_on` así:
```
%title= [@page_title, render_or_call_method_or_proc_on(self, ActiveAdmin.application.site_title)].compact.join(" | ")
```

Llama la atención que hace eso. La documentación está aquí -> https://www.rubydoc.info/gems/activeadmin-rails/MethodOrProcHelper:render_or_call_method_or_proc_on

## Rubocop y Haml-lint

Este proyecto tiene configurado rubocop y haml-lint como linters. Rubocop se encarga de archivos meramente Ruby y haml-lint las vistas escritas en Haml.

Haml-lint puede tomar como [fuente de configuraciones](https://github.com/sds/haml-lint/blob/main/lib/haml_lint/linter/README.md#rubocop) lo que se define en `.rubocop.yml` así que si se necesita excluir algo para que no revise los archivo .haml, hay que cambiar la configuración en el archivo de rubocop.

El comando de haml-lint se corre así:
```
bundle exec haml-lint app/views/**/*.haml
``

