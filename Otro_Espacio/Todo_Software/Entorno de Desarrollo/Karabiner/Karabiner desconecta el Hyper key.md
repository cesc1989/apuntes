# Karabiner se desconecta y se pierde la Hyper Key

Sobre la configuración de Hyper [[Karabiner Hammerspoon y Hyper Key para Atajos en MacOS]]

## Contexto

Me ha pasado ya dos veces que cuando arranco el macbook y trato de hacer alguna combinación con la Hyper key lo que se activa es Caps Lock.

Reiniciar Karabiner no hace nada. Desconectar el teclado no hace nada.

## Solución

Lo que me ha servido ha sido ir a Karabiner en la sección Devices y activar/desactivar alguna de las opciones de Keychron C1 Pro.

![[karabiner.settings.png]]

La primera vez que me pasó tuve que desactivar una y activar la otra. La segunda vez que me pasó fue lo contrario.

Me guié de este [issue](https://github.com/pqrs-org/Karabiner-Elements/issues/3360) para ideas pero al final aplique fue lo de activar/desactivar el _device_.