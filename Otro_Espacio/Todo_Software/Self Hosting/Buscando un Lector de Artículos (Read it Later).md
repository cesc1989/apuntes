# Buscando un Lector de Artículos (Read it Later)

## Conclusiones

No estoy del todo convencido en pasarme a Readeck o a Wallabag por las limitaciones para resaltar en móvil. Sin embargo, en Omnivore el resalto en móvil también funciona algo chueco a veces.

Muchas veces me encuentro leyendo en el móvil porque es una forma de pasar el tiempo antes de dormir o cuando estoy haciendo nada.

Las versiones de escritorio funcionan muy bien pero normalmente cuando estoy en el computador no leo muchos artículos. Lo hago más bien esporádicamente.

> [!note]
> Así que bien creo que seguiré probando todas las opciones.

# Las Alternativas

Antes para leer artículos al estilo _Read it Later_ usaba Pocket. Hoy en día, uso más Omnivore. ¿Por qué? Los resaltos y las anotaciones. En Pocket hay que pagar y me parece muy caro para la forma en que funciona (es un poco torpe en ocasiones). Por su parte Omnivore funciona mucho mejor pero de vez en cuando (porque está en desarrollo) también funciona de manera incorrecta.

Ahora que ando en el mundo del self hosting, veo que hay varias alternativas. He probado [Readeck](https://readeck.org/en/) y [Wallabag](https://wallabag.org/). A continuación, mis comentarios sobre ambos servicios para que no se me olvide esta experiencia y también para decidirme por cual seguir usando.

> [!warning]
> Ambos funcionan muy bien para leer en el navegador web en móvil pero el resaltado funciona con truco.
>
> En Wallabag, el resaltado funciona pero, luego de seleccionar un texto, hay que tocarlo y a continuación se quita la selección pero aparece el botón de crear anotación.
> 
> En Wallabag, la aplicación móvil deja mucho que desear y el botón de anotaciones tiene. Hay un [bug reciente](https://github.com/wallabag/android-app/issues/1431) sobre las anotaciones.
> 
> En Readeck, en móvil no se puede resaltar a menos qué se cambie a "Modo escritorio" y luego volver a cambiar a web móvil. Ahí funciona el resaltar pero una vez recargue la página o abra una nueva pestaña deja de funcionar.

> [!info]
> Ambos programas los probé usando Pikapods.

> [!important]
> Una de las funcionalidades principales para mí es poder ver el código del artículo. Ya que la mayoría de mis artículos serán tutoriales de programación.

# Readeck

> [!info]
> Sitio web: https://readeck.org/en/
> Escrita en Go (Stimulus + Turbo para la UI). Repo -> https://codeberg.org/readeck/readeck

De entre Readeck, Omnivore y Wallabag, este me parece del mejor UI. Bastante simple y rápida.

También parece que el extractor de texto es más potente que el de Wallabag. [Este artículo](https://garrettdimon.com/journal/posts/data-modeling-saas-entitlements-and-pricing) Wallabag no lo pudo extraer pero sí pudo Readeck.

En Readeck
![[20.readeck.extractor.png]]

En Wallabag
![[21.wallabag.extractor.png]]

En otras cosas que se pueden hacer en Readeck es poner múltiples etiquetas a los artículos y se pueden descargar versiones EPUB (aunque un poco torpe el resultado final).

## Resaltos y más

En el navegador en escritorio, los resaltos van muy bien. Me gusta que se puede resaltar y en la barra derecha las va listando. Al clicar en un resalto en el panel derecho, el lector te lleva a esa parte del texto.

Los resaltos son solo de color amarillo (contrario a Omnivore que tiene 4 tonos). No se pueden hacer anotaciones.

## Listado de enlaces externos

También tiene algo bacano que es un menú de enlaces externos. Estos son todos los enlaces que están en el artículo:

![[22.readeck.links.png]]

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

Las anotaciones en la web funcionan como anotación + resalto. Se pueden crear anotaciones vacías y solo queda el resalto.

> [!important]
> Las discusiones más recientes sobre Anotaciones son de hace varios años ([2019](https://github.com/wallabag/wallabag/issues/3839) y [2021](https://github.com/wallabag/wallabag/issues/5484)) y parece que el sistema actual es el que será por mucho.

En en el navegador Firefox en móvil funciona con truco como expliqué en el warning arriba en este documento..