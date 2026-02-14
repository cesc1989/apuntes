# Aprendizajes de los Releases

Estos son apuntes variados de los aprendizajes ganados de hacer liberaciones (releases) en proyectos. Más que nada en Luna y más ahora que trabajamos mucho más en Edge.

# Aprendizaje al lanzar Communication Preferences

*Fecha lanzamiento: 3 de Mayo de 2024.*

Del mismo modo que las tablas nuevas y migraciones se envían en un PR aparte, antes del trabajo de código, debería hacerse con las rake tasks.

Ejemplo:

- pr1: modelos, migraciones
- pr2: rake tasks
    - Hacer que Ryan ejecute en su local con dump
    - Correr en Omega
- pr3: código nuevo
    - Ya no necesito ni migraciones ni rake tasks

De esta forma, al tener tablas y datos generados, el código va a tener más probabilidades de trabajar sin errores al ser mezclado en Omega. Tampoco dependería de Ryan para correr rakes luego del despliegue.

También es clave hacer que Ryan pruebe las rakes en su local con un dump de Omega para asegurarnos que todo vaya en orden y poder corregir antes del lanzamiento.

