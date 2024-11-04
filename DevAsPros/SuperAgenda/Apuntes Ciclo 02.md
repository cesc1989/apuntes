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

## Respuesta Turbo Stream desde petición JavaScript

Para mostrar los datos del acudiente al seleccionar a un paciente preaparé un controlador Stimulus para hacer una petición mediante la función `fetch` de JavaScript:
```javascript
fetch(event) {
    const patientId = event.target.value

    fetch(`/patients/${patientId}/guardian`).then(() => {})
  }
```

De esa forma la petición se ejecutaba al cambiar la selección en el dropdown pero nada se pintaba en el HTML. Para lograr mostrar el HTML de la respuesta dos cosas me faltaban.

### Cabecera Accept en la petición

Hay que hacer que `fetch` envíe una petición con la cabecera que Turbo Stream espera:
```javascript
fetch(event) {
    const patientId = event.target.value

    fetch(`/patients/${patientId}/guardian`, {
        headers: {
          Accept: "text/vnd.turbo-stream.html"
        }
      }).then(() => {})
  }
```

Pero aún faltaba algo. Ese algo lo encontré en [este tutorial](https://www.writesoftwarewell.com/process-turbo-stream-javascript/). La respuesta está nada más visitar la página:
```javascript
.then(r => r.text())
.then(html => Turbo.renderStreamMessage(html))
```

Así queda la función en Stimulus:
```javascript
fetch(event) {
    const patientId = event.target.value

    fetch(`/patients/${patientId}/guardian`, {
        headers: {
          Accept: "text/vnd.turbo-stream.html"
        }
      })
    .then(r => r.text())
    .then(html => Turbo.renderStreamMessage(html))
  }
```

### Controlador acompaña

El controlador también debe estar configurado para pintar la respuesta debida:
```ruby
  def guardian
    patient = Patient.find(params[:id])

    render(
      turbo_stream: turbo_stream.update(
        "patient_guardian",
        partial: "patients/guardian",
        locals: { guardian: patient.guardian }
      )
    )
  end
```

Y en la vista debe haber un `turbo_frame_tag`:
```html
<div class="col-12 mb-3">
    <label>Acudiente</label>

    <%= turbo_frame_tag "patient_guardian" %>
</div>
```