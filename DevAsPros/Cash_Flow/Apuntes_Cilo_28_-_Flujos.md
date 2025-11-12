# Apuntes Ciclo 28

## Usa assets:clobber para limpiar caché pesada

Estaba haciendo un cambio de textos y resulta que se quedaba el texto "Flujo" incluso luego de cambiar el render desde JS por una versión desde HTML. Entonces quedaba algo como:

```
Flujo: Flujo: XXXX
```

Por más que buscaba y probaba no se quitaba el duplicado. No había razón evidente hasta que Claudio sugirió correr `rake assets:clobber`. Con eso se borró lo de assets en `public` y pude ver los cambios como esperaba.