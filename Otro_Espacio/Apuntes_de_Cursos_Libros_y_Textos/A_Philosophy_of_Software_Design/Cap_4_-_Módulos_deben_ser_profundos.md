# Cap.4 - Módulos deben ser profundos

## Diseño Modular

Un módulo puede componerse de dos partes: una *interfaz* y una *implementación*.

La interfaz consiste de todo lo que un desarrolla que trabaja en otro módulo debe saber para poder usar el otro módulo.


> The interface describes what the module does but not how it does it.

La implementación consiste del código que completa la promesa de la interfaz.


> A developer should not need to understand the implementations of modules other than the one he or she is working in.

Los mejores módulos son aquellos cuyas interfaces son más simples que sus implementaciones.

Si la interfaz de un módulo es más simple que su implementación, habrá muchos aspectos de este que podrán ser cambiados sin afectar a otros módulos.

## ¿Qué es una interfaz?

La interfaz de un módulo contiene dos tipos de información: formal e informal.


> The formal parts of an interface are specified explicitly in the code, and some of these can be checked for correctness by the programming language.


> The informal parts of an interface include its high-level behavior, such as the fact that a function deletes the file named by one of its arguments.

En general, si un desarrollador necesita saber una información particular para usar un módulo, entonces esa información es parte de la interfaz del módulo.

La parte informal de una interfaz solo pueden describirse usando comentarios y el lenguaje de programación no puede asegurar que la descripción sea completa o acertada.


> One of the benefits of a clearly specified interface is that it indicates exactly what developers need to know in order to use the associated module. This helps to eliminate the “unknown unknowns” problem.


## Abstracciones

Un abstracción es una vista simplificada de una entidad, omitiendo detalles sin importancia.

Las abstracciones son útiles porque facilitan pensar y manipular cosas complejas.


> The more unimportant details that are omitted from an abstraction, the better. However, a detail can only be omitted from an abstraction if it really is unimportant.


> The key to designing abstractions is to understand what is important, and to look for designs that minimize the amount of information that is important.


## Módulos profundos
![](https://paper-attachments.dropboxusercontent.com/s_A7CB480A863B8877C5585E37B5062F568FC3042211F468EF0F155D4567A85D10_1669390176796_image.png)


Los mejores módulos son aquellos que proveen muy buena funcionalidad mediante interfaces sencillas.

Los mejores módulos son profundos: tienen mucha funcionalidad oculta detrás de una simple interfaz. Un módulo profundo es una buena abstracción porque solo una pequeña parte de sus complejidad es visible a los usuarios.


## Módulos llanos/poco profundos

Un módulo llano es aquel cuya interfaz es algo compleja en comparación con la funcionalidad que aporta. Ejemplo:


    private void AddNullValueForAttribute(String attribute) {
      data.put(attribute, null)
    }


> From the standpoint of managing complexity, this method makes things worse, not better. The method offers no abstraction, since all of its functionality is visible through its interface.


## Classitis
> Classitis may result in classes that are individually simple, but it increases the complexity of the overall system.
> 
> Small classes don’t contribute much functionality, so there have to be a lot of them, each with its own interface.

