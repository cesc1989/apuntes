# Apuntes Ciclo 10 - Turbo 2 - Cash Flow

# Cómo hacer que solo se cambie la parte modificada con Turbo?

Hay que usar, en la respuesta de la acción, `turbo.stream`

Cuando se clicaba el botón “Pagado” se actualizaba la tabla de gastos y la fila que nos interesaba. Pasaban ambas cosas. Lo que quería era solo actualizar la fila.

    Started PUT "/expenditures/33" for 127.0.0.1 at 2023-08-14 23:32:47 -0500
    Processing by ExpendituresController#update as TURBO_STREAM
      Parameters: {"authenticity_token"=>"[FILTERED]", "expenditure"=>{"paid"=>"true"}, "id"=>"33"}
      
      TRANSACTION (0.1ms)  begin transaction
      ↳ app/controllers/expenditures_controller.rb:37:in `update'
      Expenditure Update (0.3ms)  UPDATE "expenditures" SET "updated_at" = ?, "paid" = ? WHERE "expenditures"."id" = ?  [["updated_at", "2023-08-15 04:32:47.907179"], ["paid", 1], ["id", 33]]
      ↳ app/controllers/expenditures_controller.rb:37:in `update'
      TRANSACTION (0.8ms)  commit transaction
      ↳ app/controllers/expenditures_controller.rb:37:in `update'
      Rendered expenditures/_expenditure.html.erb (Duration: 1.8ms | Allocations: 1307)
    [ActionCable] Broadcasting to expenditures: "<turbo-stream action=\"replace\" target=\"expenditure_33\"><template><tr style=\"background-color: #efd165\" is=\"turbo-frame\" id=\"expenditure_33\">\n  <td>\n    33\n  </td>\n\n  <td>\n    Plan Complementario SURA\n  </td>\n\n  <td>\n    $73,577\n  </td>\n\n  <td>\n    30 de Agosto\n  </td>\n\...
    Redirected to http://localhost:3000/expenditures
    Completed 302 Found in 14ms (ActiveRecord: 1.5ms | Allocations: 7610)
    
    
    Started GET "/expenditures" for 127.0.0.1 at 2023-08-14 23:32:47 -0500
    Processing by ExpendituresController#index as TURBO_STREAM
     
      ↳ app/views/expenditures/index.html.erb:42
      Rendered shared/_cashflow.html.erb (Duration: 4.5ms | Allocations: 3432)
      Rendered shared/_sidebar.html.erb (Duration: 0.2ms | Allocations: 247)
      Rendered shared/_notices.html.erb (Duration: 0.0ms | Allocations: 25)
      Rendered layout layouts/application.html.erb (Duration: 23.4ms | Allocations: 27570)
    Completed 200 OK in 27ms (Views: 23.2ms | ActiveRecord: 0.6ms | Allocations: 30550)

Esto no es lo ideal

Para lograrlo tuve que poner la fila así:

    <tr is="turbo-frame" id="<%= dom_id(expenditure) %>">

Y luego en el controlador, hacer que se reemplace dicho elemento así:

    format.turbo_stream do
      render(
        turbo_stream: turbo_stream.replace(
          @expenditure,
          partial: 'expenditures/expenditure',
          locals: { expenditure: @expenditure }
        )
      )
    end

De esa forma se hace la petición que nos interesa y solo se modifica la parte que nos interesa:

    Started PUT "/expenditures/33" for 127.0.0.1 at 2023-08-14 23:40:34 -0500
    Processing by ExpendituresController#update as TURBO_STREAM
      Parameters: {"authenticity_token"=>"[FILTERED]", "expenditure"=>{"paid"=>"true"}, "id"=>"33"}
      
      Rendered expenditures/_expenditure.html.erb (Duration: 1.1ms | Allocations: 847)
    [ActionCable] Broadcasting to expenditure_33: "<turbo-stream action=\"replace\" target=\"expenditure_33\"><template><tr style=\"background-color: #efd165\" is=\"turbo-frame\" id=\"expenditure_33\">\n  <td>\n    33\n  </td>\n\n  <td>\n    Plan Complementario SURA\n  </td>\n\n  <td>\n    $73,577\n  </td>\n\n  <td>\n    30 de Agosto\n  </td>\n\...
    No template found for ExpendituresController#update, rendering head :no_content
    Completed 204 No Content in 13ms (ActiveRecord: 1.3ms | Allocations: 7864)

Fuentes

- Este [tutorial](https://www.hotrails.dev/turbo-rails/turbo-frames-and-turbo-streams) me ayudó
- También lo que tengo en Leisure Shelf Playground


# En tablas html solo se pueden meter elementos de tablas

Buscando solución a lo anterior descubrí que no se puede usar `turbo_frame_tag` dentro de elementos de una tabla porque los rechaza.

Ver este issue https://github.com/hotwired/turbo/issues/48

Entonces la solución es:

- Usar el atributo `is` - [docs](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/is)
- Valerse de hacer un replace con turbo_stream como menciona [este comentario](https://github.com/hotwired/turbo/issues/48#issuecomment-889870346)

