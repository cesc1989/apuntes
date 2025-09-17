# Migración a Rails 7

# Pasos

1. Asegurarse de incrementar el nivel de *coverage* de la suite de pruebas
2. Estar en la versión de Ruby 2.7.1
3. Estar en la versión de Rails 6.1.4
4. Cambiar la versión de Rails en el Gemfile a la más reciente
5. Correr el comando `bundle update`
6. Correr el comando `rails app:update`
    1. Usar la opción `d` para ver las diferencias en los archivos
    2. Usar la opción `m` para hacer una mezcla sin borrar lo existente
        1. Se puede [configurar Sublime Text](https://stackoverflow.com/questions/44549733/how-to-use-visual-studio-code-as-default-editor-for-git-mergetool/44549734#44549734) como herramienta de *merge* para simplificar esta parte.
7. Revisa la [documentación aquí](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html) por cualquier pieza faltante
8. Actualiza las gemas faltantes según lo que se indique en [RailsDiff](https://railsdiff.org/6.1.4.6/7.0.0)


# Docs Relacionados

En Provider Portal → [[Errores_al_intentar_migrar_a_Rails_7]]

# Comandos

## Docker Image

Comando para genera Docker image que necesita ENVs de Sidekiq

```bash
docker image build -t pruebados --build-arg SIDEKIQ_LICENSE_KEY=xxxxx:bbbbb -f Dockerfile .
```