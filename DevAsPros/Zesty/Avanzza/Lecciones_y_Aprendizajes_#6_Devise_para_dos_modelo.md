# Lecciones y Aprendizajes #6 Devise para dos modelos

## Definir la forma de relacionar credenciales a propietarios y conductores
## Opción 1

La tabla *owners* y *drivers* tenga cada una su columna *email* y *password*. Así podrían iniciar sesión, por ejemplo, desde owners.avanzza.com y drivers.avanzza.com

## Opción 2

Crear una tabla *profile* y que esa sea la que tenga las columnas *email* y *password* y que cada *owner* y *driver* tenga un *profile*

Acerca de esta discusión: https://stackoverflow.com/questions/5437948/using-devise-for-two-different-models-but-the-same-login-form

**Opción 2 - Razón 1**
Una de las razones por la cuál la opción 2 es viable es este código que sigue:

    - if current_owner
          = link_to destroy_owner_session_path, method: :delete, class: 'link', data: { toggle: 'tooltip' }, title: 'Cerrar sesión' do
            i.fa.fa-sign-out-alt
        - elsif current_admin
          = link_to destroy_admin_session_path, method: :delete, class: 'link', data: { toggle: 'tooltip' }, title: 'Cerrar sesión' do
            i.fa.fa-sign-out-alt

Si los taxistas llegan a tener inicio de sesión web, tocaría también agregar un condicional para `current_driver`.


## Actualización

La **opción 1** es la más sencilla y la **opción 2** requeriría más configuración.
En base a eso y a las prioridades del proyecto, escogeré la:
**OPCIÓN 1**

