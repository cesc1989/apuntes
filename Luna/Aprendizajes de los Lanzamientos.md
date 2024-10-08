# Aprendizajes de los Lanzamientos

Más que nada en Edge.

# AppBlend M2 - Rails Upgrade to 7.0.7 - October 3rd

- Cuando Alpha esté trabado con los despliegues, es probablemente que haya algún problema con Kubernetes.
	- En ese caso, si se va a mandar algo a Omega, mejor estar pendiente para verificar que el despliegue se complete y no se quede trabado.
	- Más aún si es para arreglar una caída en Omega

# Release de Unify Workout Data y Email Verification - September 25th

- Prefijar los commits
	- Cuando se trabaje en Edge, prefijar los commits puede ayudar a identificarlos si toca resolver mediante git cherry-pick u otros.
- PRs en draft
	- El build se ejecuta en draft y también se puede enviar a Alfa.
	- Cuando esté todo revisado, se pide review pasándolo a "ready for review"
- Hacer dumps dos días a la semana
	- Es mucho más cómodo trabajar con la BD local en Edge
	- Hacer dumps los Lunes y Miércoles para evitar conflictos con migraciones
	- Mejor ir haciendo la costumbre porque luego estarán todos los backends en el mismo proyecto