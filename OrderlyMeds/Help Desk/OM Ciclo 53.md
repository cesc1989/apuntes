# OM Ciclo 53

> [!Info]
> Del Jueves 13 de Agosto hasta el Miércoles 26 de Agosto.

## Caso OM-10978 - Medication Request en Cancelled 🟢

Etiquetas: #om_missing_previous_medication 

Es el mismo caso de:
> We're having trouble finding your previous medication and are unable to proceed with your order. Please contact support for assistance.

El caso es que ya tiene Medication Request importado pero queda con estado "cancelled" porque el Script en Ontraport tiene ese estado.

La solución aquí es cambiar el estado a "Completed" porque no hay más forma de corregirlo. Esto aunque puede que haya disparidad si se revisa.

## Caso OM-11055 - Resubmit para quitar Add-On 🟢

Etiquetas: #om_salesforce_resubmit #om_salesforce_resubmit_without_addon 

Dice:
> Initially ordered Casa Tirzepatide/B12 10mg/12.5mg (2 month), Casa Sermorelin 1mg/ml injection (2 month). She then reported that the sermorelin add-ons was placed by mistake so she has been refunded for it.
>
> We are unable to resubmit the script to Tirzepatide/B12 without the add-ons in SF but unable to do so. Please resubmit.

Habían sugerido otro MedId pero cuando se iba a completar el Resubmit to MSO seguía saliendo el Add-On en la vista previa. Así que al final la solución es crear un nuevo Member Period para que el CX pueda elegir la medicina sin el add-on.

![[OM_11055.png]]