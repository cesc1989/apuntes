# Apuntes Apex - Ciclo 002

# Mejoras a scripts de despliegue

Quiero fortalecer los scripts de despliegue. He visto muchos casos donde completan con éxito pero en los logs hubo fallos.

## Usando pipefail en scripts de despliegue

DeepSeek recomendó esto:
```bash
#!/bin/bash

set -euo pipefail
```

En cada script de despliegue. Pero después de varios problemas tocó solo dejarlo en:
```bash
set -eo pipefail
```

### Error de chruby `PREFIX: unbound variable` 🟢

> [!Note]
> Hay problema de chruby con `set -u`
>
> Ver: https://github.com/postmodern/chruby/issues/417

Al poner esa línea en el script `003_after_deploy.sh` da este error en github actions:
```
/usr/local/share/chruby/chruby.sh: line 4: PREFIX: unbound variable
```

> [!Important]
> La solución fue no usar `-u` porque chruby no juega bien con eso.

Se han probado varias cosas pero nada parece resultar. Probé todo esto.

**Exportar PREFIX**
```bash
export PREFIX="${PREFIX:-/usr/local}"

. /home/ubuntu/.profile
```

**Desactivar -u y reactivarlo**
```bash
set +u
. /home/ubuntu/.profile
set -u
```

Combinar las anteriores
```bash
set +u
export PREFIX="${PREFIX:-/usr/local}"
. /home/ubuntu/.profile
set -u
```

Set después de cargar el `.profile`
```bash
. /home/ubuntu/.profile

set -eo pipefail
```

### Detalle Importante: scripts no llegaban al servidor 🔑

> [!Danger]
> El problema es que todos los cambios que mandé nunca fueron ejecutados porque el servidor se quedó con los archivos donde el despliegue fallaba.
> 
> Cada vez que se corría el action del despliegue, se estaban ejecutando los scripts dañados.

Si agrego este paso al despliegue puedo copiar los scripts antes de ejecutarlos:
```yml
- name: Upload scripts to VPS
	run: |
		scp -r scripts/* supermenu_cloud:~/supermenu/deployments/api-release/scripts/  
```

> [!Warning]
> Esto tiene un problema. Es que modifica los archivos y causa que el pull falle porque "hay cambios" en el repo git que tiene el servidor.


# Usar Nested Form con Stimulus Component

Etiquetas: #supermenu

Este es el componente: https://www.stimulus-components.com/docs/stimulus-rails-nested-form/

Las instrucciones de uso son sencillas y fácil de seguir. Todo funcionó normal excepto un detalle.

## Quitar elemento existente del nested form

Cuando se agrega un nuevo elemento y se clica el botón de quitar, este se quita de la vista.

Sin embargo, cuando se clica el botón quitar de un elemento existente (cargado de la BD), no se quita. Esto pasa porque la funcionalidad del componente (`RailsNestedForm#remove`) trabaja de dos formas diferentes.

Cuando el elemento es nuevo, busca leer el data attribute `newRecord`. Cuando no, tira a ocultar el elemento con `display: none`:
```js
if (wrapper.dataset.newRecord === "true") {
      wrapper.remove()
    } else {
      wrapper.style.display = "none"
```

[Fuente](https://github.com/stimulus-components/stimulus-components/blob/master/components/rails-nested-form/src/index.ts#L32C5-L35C37)

En el caso de Super Menú así se define el elemento:
```html
<div class="nested-form-wrapper variant-row d-flex gap-2 mb-2 align-items-end" data-new-record="<%= vf.object.new_record? %>">
 # ...
</div>
```

En el caso de Super Menú esto no funciona porque se usa la clase de Boostrap `d-flex`. Esta clase pone un `!important`:
```css
.d-flex {
  display: flex !important;
}
```

Dicho important invalida la clase inline que agrega RailsNestedForm así que no se oculta.

La solución es extender el componente con un controlador Stimulus del proyecto y modificar el método `remove` para que haga algo con respecto a los elementos existentes.

### Extender RailsNestedForm

Al final así lo resolví, con la ayuda de DeepSeek:
```js
import RailsNestedForm from "@stimulus-components/rails-nested-form"

export default class extends RailsNestedForm {
  remove(e) {
    e.preventDefault()

    const wrapper = e.target.closest(this.wrapperSelectorValue)

    if (wrapper.dataset.newRecord === "true") {
      wrapper.remove()
    } else {
      /* Esto es lo diferente al componente original */
      wrapper.classList.add("d-none")

      const input = wrapper.querySelector("input[name*='_destroy']")

      if (input) input.value = "1"
    }
  }
}
```

Funciona todo al pelo. Agregar y borrar elementos del formulario.