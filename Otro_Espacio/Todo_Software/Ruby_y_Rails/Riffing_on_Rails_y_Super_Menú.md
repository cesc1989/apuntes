# Riffing on Rails y Super Menú
Repo https://github.com/kaspth/riffing-on-rails

> It's done in a blank file where you keep adding, removing or refining a sketch of Ruby code to prove out your design.
> 
> This combines a focus on top-down design with listening to what the budding implementation is telling you — and if the implementation reveals an issue with your design, you now know you need to change your design early.


# Apuntes de las sesiones de Kasper
## Apuntes Sesión “First Ruby Friend” y “RSS”

**First Ruby Friend**
A la hora del vídeo(+/-), Kasper muestra como ejecuta todo el diseño que hizo desde la consola.


> Se ve muy bueno eso para probar ideas sin tener que crear una app nueva o ir más allá del cuaderno.

**RSS**
Kasper trata de detallar lo más posible el modelo en general. No gasta detalles en campos en especifico ni tampoco en detalles al grano como "si un feed es atom o rss". Eso queda para la implementación.


> En una sesión Riffing lo importante es definir un modelo de datos y poder probarlo.

Kasper incluso llega hasta detallar cómo podría ser una vista y rutas. La vista no crea un archivo sino que escribe el código que emula una vista. Lo mismo con las rutas.

La clave aquí es que ya uno sabe cómo funciona Rails y uno se puede ahorrar la implementación del framework para ir directo al código que nos concierne y escribirlo tal cual cómo podría ser.

Es una forma de prototipar más rápida sin tanto gasto y fácilmente trasladable a un proyecto Rails.

En el tiempo 1:39:52 muestra como usa CurrentAttributes.

En el QA, Keita pregunta si sería más fácil empezar por el controlador. Kasper dice que sí ya que el controlador involucra muchas partes y se puede tener una visión más amplia.

Jeremy dice que depende de que tan claro sea el modelo de datos.

## Apuntes sesión “Mix Playlist”

En esta sesión pude ver cómo Jeremy y Kasper le daban buen tiempo a definir y llegar a un modelo satisfactorio. Debatieron forma de acceso a los datos, el nombre de las clases y de las tablas y **dada a la facilidad del entorno “Riffing” era fácil cambiar de nombres sin tener que incurrir en un gasto de esfuerzo**.

Esto último es clave porque cambiar de nombre de una tabla una vez que está creada y en uso es más costoso para uno.

Ejemplo de eso es ahora en Patient Self Report que debo cambiar la table “patients”. Obviamente, hace 4 años no tenía esta visibilidad pero es un ejemplo de como el entorno “Riffing” permite hacer esos cambios bruscos y probar con poco gaste de tiempo/energía.

# Riffing con Super Menú

Super Menú es lo que estoy llamando la idea de hacer una app para gestionar menús de restaurantes como Cluvi u Ola Click.

Hice el modelo con una sesión de Riffing y vaya que me gustó. Aquí el resultado: https://github.com/cesc1989/riffing/blob/main/supermenu.rb

Lo que más me gustó fue poder cambiar el modelo varias veces sin incurrir en gasto de migraciones y comandos de Rails, lanzar servidor, probar mediante las vistas o pruebas. Simplemente escribí unos modelos ligeros, migraciones ligeras.

Muy rápido probar e iterar el modelo.

