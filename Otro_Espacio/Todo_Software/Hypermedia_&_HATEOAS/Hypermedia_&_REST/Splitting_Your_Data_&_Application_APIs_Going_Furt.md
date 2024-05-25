# Splitting Your Data & Application APIs: Going Further - Apuntes
Ya se mencionó el tener dos APIs: una genérica de datos y otra de aplicación.

Si se hace esto, la de datos debe ser estable la mayor parte del tiempo y la de aplicación puede romperse más a discreción.

En el caso de la API de aplicación, conviene que esta devuelva Hypermedia y aprovechar librerías como HTMX o Hotwire en el caso de Ruby on Rails.

**Necesidades API genérica de datos**

![](https://paper-attachments.dropboxusercontent.com/s_884A425573EB006CA8CD0088801DFF6178C74EE3A676C7BF746C446BD56BBA66_1700002821261_imagen.png)


**Necesidades de una API de aplicación**

![](https://paper-attachments.dropboxusercontent.com/s_884A425573EB006CA8CD0088801DFF6178C74EE3A676C7BF746C446BD56BBA66_1700002880879_imagen.png)


En este sentido, una API de aplicación sería algo así:

![](https://paper-attachments.dropboxusercontent.com/s_884A425573EB006CA8CD0088801DFF6178C74EE3A676C7BF746C446BD56BBA66_1700002953824_imagen.png)


Tener una aplicación de API ayuda en el sentido que se puede devolver mucha más información a una página sin tener que hacer muchas peticiones. No habría tanto problema con cambiar cosas pequeñas cada que cambie el diseño. Se podría hacer una sola consulta SQL para obtener todos los datos necesarios.

Todo eso se podría lograr valiéndose de usar Hypermedia.

## Conclusión
> switch to a hypermedia application API (which really just means “use HTML, like you used to”) then you get all of the benefits of the REST-ful web model (simplicity, reliability, etc.) and of server-side rendering in mature web frameworks (caching, SQL tuning, etc.)

