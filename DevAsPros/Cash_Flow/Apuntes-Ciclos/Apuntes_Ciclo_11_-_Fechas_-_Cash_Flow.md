# Apuntes Ciclo 11 - Fechas - Cash Flow

# Sobre los gastos de X fechas que no aparecían

Hay un gasto que es el día 30 de cada mes

    { name: 'Plan Complementario SURA', amount: 73_577, day_of_month: '30', recurring: true }

Para Septiembre, cuando ejecuté el comando para crear los gastos recurrentes esa no se creó y luego cuando lo hice no se mostraba en la lista:

![](https://paper-attachments.dropboxusercontent.com/s_5A418457C59A6E0FE13B392F9E7154ED75E100DB74AEEAECEA8E8A287980C6C0_1694574457815_gasto.fecha.30.png)


Al revisar los gastos para la categoría de “SURA”, veía esto:

    [
      #<Expenditure:0x00007fd429e1ef38
      id: 23,
      description: "",
      amount: 73577,
      spent_at: Sun, 30 Jul 2023 19:00:00.000000000 -05 -05:00,
      category_id: 48,
      user_id: 1,
      created_at: Mon, 17 Jul 2023 11:22:40.750016000 -05 -05:00,
      updated_at: Mon, 17 Jul 2023 11:22:40.750016000 -05 -05:00,
      paid: false,
      recurring: false>,
      
     #<Expenditure:0x00007fd429dd7908
      id: 79,
      description: nil,
      amount: 73577,
      spent_at: Tue, 29 Aug 2023 19:00:00.000000000 -05 -05:00,
      category_id: 48,
      user_id: 1,
      created_at: Tue, 01 Aug 2023 11:12:00.500681000 -05 -05:00,
      updated_at: Tue, 29 Aug 2023 13:16:55.503405000 -05 -05:00,
      paid: true,
      recurring: true>,
      
     #<Expenditure:0x00007fd429dd7778
      id: 165,
      description: nil,
      amount: 73577,
      spent_at: Fri, 29 Sep 2023 19:00:00.000000000 -05 -05:00,
      category_id: 48,
      user_id: 1,
      created_at: Fri, 01 Sep 2023 10:58:03.940785000 -05 -05:00,
      updated_at: Fri, 01 Sep 2023 15:42:08.904452000 -05 -05:00,
      paid: false,
      recurring: true>
    ]

El gasto 165 existía y tenía bien la fecha pero no aparecía al aplicar los filtros correspondientes.

## ¿Qué ocurre?

Ni idea. Lo que hice para solventar fue modificar su fecha a un día antes y se solventó. Hay un error que no logro comprender al guardar la fecha de gasto y convertirlo a la vista.

*NOTA: esto sigue pasando.*

# Comparando fechas

La clase [DateTime::Calculations](https://api.rubyonrails.org/classes/DateAndTime/Calculations.html) de rails provee de métodos para simplificar o hacer comparaciones de fechas de forma más semántica.

En este caso usé los métodos `#after?` o `#before?`

    show_from_date.after?(Date.today)

Este [artículo de Boring Rails](https://boringrails.com/tips/rails-date-before-after) trata el tema.

# Matchers de Capybara y la contraparte en RSpec

Capybara RSpec matchers → [https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/RSpecMatchers](https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/RSpecMatchers)

Capybara Node Matchers → [https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Node/Matchers](https://rubydoc.info/github/teamcapybara/capybara/master/Capybara/Node/Matchers)

