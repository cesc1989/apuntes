# Buscando un Lector de Artículos (Read it Later)

Antes para leer artículos al estilo _Read it Later_ usaba Pocket. Hoy en día, uso más Omnivore. ¿Por qué? Los resaltados y las anotaciones. En Pocket hay que pagar y me parece muy caro para la forma en que funciona (es un poco torpe en ocasiones). Por su parte Omnivore funciona mucho mejor pero de vez en cuando (porque está en desarrollo) también funciona de manera incorrecta.

Ahora que ando en el mundo del self hosting, veo que hay varias alternativas. He probado [Readeck](https://readeck.org/en/) y [Wallabag](https://wallabag.org/). A continuación, mis comentarios sobre ambos servicios para que no se me olvide esta experiencia y también para decidirme por cual seguir usando.

> [!bug]
> Ambos programas funcionan muy bien para leer en el navegador en Móvil pero no funciona el resaltado.
> 
> En Wallabag, la aplicación móvil deja mucho que desear y el botón de anotaciones tiene. Hay un [bug reciente](https://github.com/wallabag/android-app/issues/1431) sobre las anotaciones.

> [!info]
> Ambos programas los probé usando Pikapods.

> [!important]
> Una de las funcionalidades principales para mí es poder ver el código del artículo. Ya que la mayoría de mis artículos serán tutoriales de programación.

# Readeck

Sitio web: https://readeck.org/en/

De entre Readeck, Omnivore y Wallabag, este me parece del mejor UI.

# Wallabag

> [!info]
> Escrita en PHP (Symfony). Repo -> https://github.com/wallabag/wallabag

La UI me parece bastante simple, sin embargo, el lector de artículos es muchísimo mejor que en otras alternativas que he probado en cuanto a mostrar código.

![[10.wallabag.code.highlight.png]]

Esto es algo que no he encontrado en ninguna de las otras opciones que he probado, incluyendo Omnivore (el cual es decente).

De resto me parece muy bien. Sirve para lo que necesito que es "guardar artículos para leer después", tengo resaltos ilimitados y lo mejor es que corre en un Pod bastante sencillo:

![[10.wallabag.pikapod.png]]

Finalmente, tiene posibilidad de exportar los artículos a varios formatos incluyendo PDF y EPUB.

## Anotaciones

Las anotaciones en la web funcionan solo en Escritorio. Funcionan como anotación + resalto. Se pueden crear anotaciones vacías y solo queda el resalto.

> [!important]
> Las discusiones más sobre Anotaciones son de hace varios años ([2019](https://github.com/wallabag/wallabag/issues/3839) y [2021](https://github.com/wallabag/wallabag/issues/5484)) y parece que el sistema actual es el que será por mucho.

En en el navegador Firefox en móvil el botón anotaciones no existe prácticamente. No se puede usar esta característica al parecer.