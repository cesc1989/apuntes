# Apuntes Ciclo 02 - Variados - Coshi Notes

# Usando Bootstrap

Bootstrap ofrece clases para ocultar elementos según el breakpoint. Más fácil que usar mediaqueries para algunos casos.

Documentación de las clases aquí → https://getbootstrap.com/docs/5.3/utilities/display/#hiding-elements



# Ruta o URL para formulario de recursos anidados

En Coshi Notes tengo los modelos Canales y Temas anidados:

    resources :channels do
      resources :themes
    end

Para que el form de crear y actualizar, el parámetro del helper `form_with` debe escribirse así:

    form_with(model: @activity, url: [@trip, @activity])

Así lo vi en [Stack Overflow](https://stackoverflow.com/a/46919792/1407371).


# Sidebar fijo y contenido principal scrollable

Para lograr que el sidebar quede fijo el HTML debe estar de esta forma

    <!-- Side navigation -->
    <div class="sidenav">
      <a href="#">About</a>
      <a href="#">Services</a>
      <a href="#">Clients</a>
      <a href="#">Contact</a>
    </div>
    
    <!-- Page content -->
    <div class="main"></div>

Y hay que estilizarlo así:

    .sidebar {
      position: static;
      top: 0;
      height: 100%;
    }

Visto en https://www.w3schools.com/howto/howto_css_fixed_sidebar.asp

