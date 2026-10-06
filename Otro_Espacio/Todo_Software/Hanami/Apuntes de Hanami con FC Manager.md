# Apuntes mientras hago FC Manager junto con Deep Seek

La parte inicial del modelo de datos, migraciones y relaciones lo hice yo. Luego le cedí más control a Deep Seek para migrar la maquetación y estilos desde el prototipo al proyecto. Ha hecho varias cosas raras entonces aquí apunto lo que aprendo de todo eso.

Esto es lo que quiero revisar:

- Contracts en create action
- PlayerForm en edit action
- handle(*, response) en new action
- upsert de ROM en PlayerRepo
- Forms:
    - se pasa values en el partial
    - se pasa el method y el cancel paths
- Views:
    - layout = app
    - expose sin usar repos     
- Helpers:
    - Hay bastante codigo. Revisar para tener claridad.
- Lib:
    - PlayerCatalog y TraitsCatalog