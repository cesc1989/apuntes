# Apuntes Ciclo 05 - Pre Lanzamiento - Cash Flow

# title tag no mostraba bien el contenido en móvil

Resulta que estaba roto porque había espacios y parece que espera estar en una misma línea

![](https://paper-attachments.dropboxusercontent.com/s_50AD1546514BF1E55164147498CAC55648DEAA29F7CF1ED75587B3E508D8D18A_1689455269573_imagen.png)



# Precargar valor en date_field

Aquí encontré las diferencias entre `date_field y date_select` → [https://blog.corsego.com/datetime-select-or-field](https://blog.corsego.com/datetime-select-or-field)

Docs sobre `date_field` → [https://apidock.com/rails/ActionView/Helpers/FormHelper/date_field](https://apidock.com/rails/ActionView/Helpers/FormHelper/date_field)

Sí hay que cargar el atributo value con el formato YYYY-MM-DD:

    <%= form.date_field :spent_at, class: 'form-control', value: @expenditure.new_record? ? Date.today : @expenditure.spent_at.strftime("%Y-%m-%d") %>


