# Apuntes Ciclo 09 - Formularios - Cash Flow

# Limpiar los campos de un formulario

Se hace así:

    <%= button_tag "Reset", type: :reset %>

Y así quedó en Cashflow:

    <%= form.button 'Limpiar', type: :reset, class: 'btn btn-warning mb-2' %>

Visto en [Stack Overflow](https://stackoverflow.com/a/14311767/1407371).

# Formas para crear una lista de selección con los meses del año

Forma 1:

    <%= f.date_select :date,  { :discard_day => true} %>

[Docs](http://api.rubyonrails.org/classes/ActionView/Helpers/DateHelper.html#method-i-date_select).

Forma 2:

    <% the_months = (Date.new(2020,1,1)..Date.new(2020,12,31)).select { |d| d.day == 1 } %>
    <%= f.select :start_date, the_months.map { |d| [d.strftime('%b %Y'), d] } %>

La cual usé en Cashflow en la sección Análisis:

    <%= f.select :month, months_of_the_year, { selected: selected_month_in_analitics }, { class: 'form-control' } %>

El helper:

      def months_of_the_year
        months_of_the_year = (Date.today.beginning_of_year..Date.today.end_of_year).select { |d| d.day == 1 }
    
        months_of_the_year.map do |date|
          [
            l(date, format: :month_name),
            date
          ]
        end
      end

Visto en [Stack Overflow](https://stackoverflow.com/a/66217332/1407371).

