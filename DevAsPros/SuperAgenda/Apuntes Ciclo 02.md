# Apuntes Ciclo 02

## Capturar momento después que un form sea submiteado

Para ocultar el panel derecho donde está el formulario de nueva cita después de guardar hice un controlador Stimulus
```javascript
import { Controller } from "@hotwired/stimulus"

// Connects to data-controller="appointment-form"
export default class extends Controller {
  connect() {
    // console.log("form-controller#connect")
  }

  submit(event) {
    this.element.reset()

    const appointmentFormPanel = document.getElementById("appointment-form-panel")
    const monthCalendarPanel = document.getElementById("month-calendar-panel")

    appointmentFormPanel.classList = "closing d-none"
    monthCalendarPanel.classList = "col-12"
  }
}

```

Quería hacer el cierre del panel después del submit. Si lo hacía en el evento submit se vería muy mal todo.

En la [referencia de Turbo](https://turbo.hotwired.dev/reference/events#forms) está el evento que sirve para eso, `turbo:submit-end`:
```ruby
<%= form_with(
  model: @appointment,
  local: true,
  data: {
    turbo: true,
    controller: "form",
    action: "turbo:submit-end->form#submit"
  },
  tabindex: 0
) do |form| %>
```