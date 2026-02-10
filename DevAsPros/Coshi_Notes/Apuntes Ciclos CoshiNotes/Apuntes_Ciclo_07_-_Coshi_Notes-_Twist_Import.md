# Apuntes Ciclo 07 - Coshi Notes- Twist Import
Por alguna razón, la API de twist no me dejaba hacer peticiones desde Postman ni desde http.rb así que me tocó usar curl para hacer las peticiones y guardar los resultados en archivos JSON que luego use para importar los datos.

Esto me dejó pensando en que en una eventual cerrada de Twist, la exportada de los datos habría sido un suplicio. Que bien que me salgo ahora.

# Usando Rsync para sincronizar archivos entre local y servidor

En este [tuto](https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories#understanding-rsync-syntax) hay suficiente info para usar este comando.

Así quedó el comando para sincronizar los archivos con los datos al servidor:

    rsync -anv --exclude .DS_Store ~/Documents/coshinotes/twist-import cashflow:

Con las banderas `nv` hacer un dry-run y de forma verbosa. Sirve para ver qué se está enviando al servidor.

Para la ejecución definitiva, se quitan esas dos banderas. Solo debe quedar `-a`.

## Enviando la carpeta vs los contenidos de la carpeta

Para enviar toda la carpeta y contenidos, es de esta forma:

    ~/Documents/coshinotes/twist-import

Para enviar *solo* los contenidos de la carpeta, sería así:

    ~/Documents/coshinotes/twist-import/

Gran diferencia.

# Leyendo archivo CSV en Bash

Se hace así:

    while IFS="|" read -r theme_name twist_id
    do
      echo "Tema: $theme_name"
      echo "TID: $twist_id"
    done < <(tail -n +2 ~/Documents/coshinotes/twist-integration/temas.csv)

Hay que pasar lo que imprime el comando `tail` empezando desde la línea 2.

`IFS=``"``|``"` es el separador de líneas del CSV.

Nota los espacios entre la definición de cada columna.

Los signos menor van separados:

    done < <(ta

Visto en [How To Geek](https://www.howtogeek.com/826549/parse-csv-data-in-bash/).

# Borrando comentario usando Turbo Stream

Resulta que aquí bastó con responder con `turbo_stream` en la acción del controlador.

    def destroy
      comment = current_user.comments.find(params[:id])
      comment.destroy
    
      respond_to do |format|
        format.html {}
        format.turbo_stream do
          render turbo_stream: turbo_stream.remove(comment)
        end
      end
    end

A diferencia de al crear que solo bastó con lo que está en el modelo.

Iba a usar el [callback](https://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html) `after_destroy` junto con el [macro](https://github.com/hotwired/turbo-rails/blob/main/app/models/concerns/turbo/broadcastable.rb#L175) `broadcast_remove` pero por lo anterior no fue necesario.

Finalmente, así quedó el enlace para borrar, que haga la petición via Turbo y muestre el modal de confirmación:

    <%= link_to comment_path(comment), class: 'delete_comment', data: { turbo_method: :delete, turbo_confirm: '¿Borrar comentario?' } do %>
        <i class="fas fa-trash"></i>
    <% end %>

Enlaces:

- En [esta respuesta](https://stackoverflow.com/a/75657122/1407371) el tipo explica el comportamiento para responder con HTML o Turbo Stream.
- Documentación de [link_to](https://apidock.com/rails/ActionView/Helpers/UrlHelper/link_to)

