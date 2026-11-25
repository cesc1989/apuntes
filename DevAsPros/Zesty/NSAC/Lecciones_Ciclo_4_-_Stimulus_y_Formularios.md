# Lecciones Ciclo 4 - Stimulus y Formularios

Para el formulario de respuesta en el PQR, tenía el detalle de que necesitaba mostrar el objeto Flash. Sin embargo, con la respuesta normal no se mostraba:

```ruby
format.html { redirect_to pqr_path(pqr), notice: 'Respuesta enviada' }
```

Porque el formulario está envuelto en un `turbo_frame_tag`. Entonces, para que se vea el Flash había que responder con un `turbo_stream`.

```ruby
flash.now[:success] = 'Respuesta enviada de manera satisfactoria'
format.turbo_stream do
  render(
    turbo_stream: turbo_stream.update("flashes", partial: "shared/notices")
  )
end
```

Pero de esa forma no se limpiaba el formulario porque no había redirección. Así que para limpiarlo y de paso mover el scroll hacía arriba para ver el mensaje tocó usar Stimulus.

```javascript
// app/javascript/controllers/response_form_controller.js

import { Controller } from "stimulus"

export default class extends Controller {
  reset() {
    this.element.reset()
  }

  scrollToTop () {
    window.scrollTo(0, 0)
  }
}
```

y en el formulario:

```ruby
<%= form_with(
      model: response,
      url: responses_path,
      id: dom_id(response),
      local: true,
      data: { controller: 'response-form', action: 'turbo:submit-end->response-form#reset turbo:submit-end->response-form#scrollToTop' }
    ) do |f| %>
```

Enlaces:

- Cómo [mover el scroll hacía arriba](https://discuss.hotwire.dev/t/how-to-return-the-user-to-the-top-of-the-page-after-a-422-unprocessable-entity-response/2881) con Stimulus
- Cómo [limpiar el formulario](https://discuss.hotwire.dev/t/turbo-form-not-resetting-with-stimulus-2-0-controller/1831) con Stimulus
- Convención de [nombre de controladores](https://stimulus.hotwire.dev/reference/controllers#naming-conventions) en Stimulus
- Sobre [varias acciones](https://stimulus.hotwire.dev/reference/actions#multiple-actions) en un mismo `data-action`

