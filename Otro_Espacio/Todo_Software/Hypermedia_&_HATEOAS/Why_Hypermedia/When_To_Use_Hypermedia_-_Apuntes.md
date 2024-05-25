# When To Use Hypermedia? - Apuntes

# ¿Cuándo queda bien usar Hypermedia?
- Si tu aplicación es más que nada imágenes y texto.
- Si tu aplicación es un CRUD: formularios y listados.
- Si la UI de tu app es anidada y las actualizaciones se dan en bloques concretos.
    - Ejemplo, tener tres secciones donde las actualizaciones son independientes entre cada una.
- Si tu app necesita “deep links” o buena carga inicial.


# ¿Cuándo no queda bien usar Hypermedia?
- Si tu aplicación tiene muchas interdependencias dinámicas
    - Ejemplo: Google Sheets, Google Maps
- Si tu aplicación necesita funcionalidad offline
- Si el estado de tu UI cambia muchas veces
    - Mismos ejemplos de arriba
- Si tu equipo no lo prefiere
    - Ejemplo: un equipo de devs frontend JavaScript React

