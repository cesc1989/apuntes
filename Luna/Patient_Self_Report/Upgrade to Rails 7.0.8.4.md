# Upgrade a Rails 7.0.8.4

## Error de OpenStruct por WickedPDF

Este error cuando corría las pruebas:
```bash
Failure/Error: OpenStruct.new(valid?: true, errors: @errors)

      NameError:
        uninitialized constant Answers::ValidateValue::OpenStruct

                OpenStruct.new(valid?: true, errors: @errors)
                ^^^^^^^^^^
```

Aparecía en muchas partes donde se usaba OpenStruct pero antes sí funcionaba, ¿qué cambió? N idea.

En todo caso, hubo gente en la misma situación como se ve en [este post](https://discuss.rubyonrails.org/t/rails-7-0-8-1-app-not-loading-due-to-nameerror-uninitialized-constant-wickedpdf-openstruct/85768) en el foro de Rails. La solución estaba en requerir la librería `ostruct`.

En su caso era con WickedPDF:
```
NameError: uninitialized constant WickedPdf::OpenStruct
```

A mí no me pasó así pero eso me dio una pista y luego seguí el [issue enlazado](https://github.com/mileszs/wicked_pdf/pull/1110) en post. Es un PR donde se requiere la librería `ostruct` en la gema WickedPDF. Probé actualizando la gema a que apunte a master y se arregló el problema.

```ruby
# Gemfile

gem "wicked_pdf", github: "mileszs/wicked_pdf", branch: "master"
```

### Cambió la forma de configurar wicked_pdf

En la versión más reciente, se cambió la forma de configurar esta gema.

```diff
- WickedPdf.config = {
-   exe_path: executable_path,
-   lowquality: true,
-   viewport_size: '1024x768'
- }
+ WickedPdf.configure do |c|
+   c.exe_path = executable_path,
+   c.lowquality = true,
+   c.viewport_size = '1024x768'
+ end
```