# Apuntes de HTMX y Rails

## Enlaces
- [Learning AJAX made easy - Build an app with htmx](https://dev.to/libsyz/learning-ajax-made-easy-build-an-app-with-htmx-gie)
- Para generar gifs a partir de vídeos - [gist](https://gist.github.com/dergachev/4627207)
- Generar gifs [en línea](https://cloudconvert.com/mov-to-gif)
## Instalación

La instalación con Importmaps no pareció funcionar solo con el comando:

    ./bin/importmap pin htmx.org

Así que instalé mediante script tag y CDN:

    <script src="https://unpkg.com/htmx.org@1.9.3" integrity="sha384-lVb3Rd/Ca0AxaoZg5sACe8FJKF0tnUgR2Kd7ehUOG5GCcROv5uBIZsOqovBAcWua" crossorigin="anonymous"></script>
# En Rails

Que quede claro que HTMX es para todo aquello donde pase una interacción cliente-servidor:

- Pedir un listado de artículos
- Enviar un formulario
- Traer información de algo

Pero con Rails, la acción de un controlador trae todo un layout así que cuando tenía algo como:

    # app/views/moviews/_movie.html.erb
    
    <%#= link_to movies_path(movie.id) do %>
    <a hx-get="/movies/<%= movie.id %>">
      # código
    </a>
    <%# end %>
    

Se cargaba toda la vista “show” de movies en el mismo elemento HTML donde está el enlace. Cargaba absolutamente todo.

![](https://paper-attachments.dropboxusercontent.com/s_AE99238B19500D99A6DD907DBA44A16AFB3B4359B3F21B48FA1D3D22A5DB0864_1690145441675_imagen.png)


Usando la gema [rails-htmx](https://github.com/guilleiguaran/rails-htmx) es que se pudo controlar ese proceso ya que está intercepta la petición y revisa si hay una cabecera `HX-REQUEST`. Si la hay, no devuelve layout.

# Objetivos de HTML

[Docs](https://htmx.org/docs/#targets).

Si lo que se quiere mostrar debe verse en otra parte de la página, hay que indicarlo con el atributo `hx-target`. Sin eso, se vería así en este caso:

![](https://paper-attachments.dropboxusercontent.com/s_AE99238B19500D99A6DD907DBA44A16AFB3B4359B3F21B48FA1D3D22A5DB0864_1690145598336_imagen.png)


Para nuestro ejemplo, queremos que se reemplace todo lo que se ve en el contenedor con lo que viene en la vista show. Ponemos el atributo `hx-target` en el padre más cercano:

    <!-- <div class="px-6 py-4 flex-1"> -->
    <div class="px-6 py-4 flex-1" hx-target="this">
      <%= yield %>
    </div>

Resultado:

![reemplaza al elemento padre (gif)](https://paper-attachments.dropboxusercontent.com/s_AE99238B19500D99A6DD907DBA44A16AFB3B4359B3F21B48FA1D3D22A5DB0864_1690146067025_htmx.target.gif)



# Actualizar el historial de navegación

[Docs.](https://htmx.org/docs/#history)

De la forma anterior, cuando se da clic al elemento la URL se mantiene en la lista de movies:

![](https://paper-attachments.dropboxusercontent.com/s_AE99238B19500D99A6DD907DBA44A16AFB3B4359B3F21B48FA1D3D22A5DB0864_1690146198519_imagen.png)


Para hacer que cambie a la URL del detalle, hay que usar el atributo `hx-push-url="true"`:

    <a hx-get="/movies/<%= movie.id %>" hx-push-url="true">
      <!-- código -->
    </a>

Resultado:

![cambia el historial del navegador (gif)](https://paper-attachments.dropboxusercontent.com/s_AE99238B19500D99A6DD907DBA44A16AFB3B4359B3F21B48FA1D3D22A5DB0864_1690146675822_history.gif)



