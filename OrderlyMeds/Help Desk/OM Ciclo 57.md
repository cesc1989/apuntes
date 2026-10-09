# OM Ciclo 57

> [!Info]
> Del Jueves 8 al Miércoles 21 de Octubre.

## Casos de problemas de Sermorelin en MedPicker 🟢

Etiquetas: #om_bundle_issue #om_bundle_manually_rejected 

Esto es lo mismo que pasó en OM-11617 -> [[OM Ciclo 56#Detalle de OM-11617 - Not at the Pharmacy - Bundle Rejected 🟢]]

Casos:

- OM-11638
- OM-11779

Capturas:

<figure>
    <img src="./attachments/OM-11638.png" />
    <figcaption>Caso OM-11638</figcaption>
</figure>


<figure>
    <img src="./attachments/OM-11779.png" />
    <figcaption>Caso OM-11779</figcaption>
</figure>


### Solución: Tenían que corregir el MedPicker

Cuando le fui a escribir a Sarah se me dio por revisar antes y pude hacer los resubmits.

## Caso OM-11848 - Unable with Current Medication 🟡

Etiquetas: #om_unable_to_continue_with_current_medication

Este error cuando la persona va a seleccionar tratamiento:
![[OM_11848.png]]

Claudio dice que pasa porque:
- para mostrarle su tratamiento actual (Tirzepatide inyectable), la app le pide a MedPicker las opciones para ella y luego se queda solo con las que existen en nuestra tabla de medicamentos (`Salesforce::Medication`).
- MedPicker sí le recomendó 3 opciones de Tirzepatide 12.5mg de PerfectRX, pero ninguna se importó nunca a esa tabla, así que la app las descarta. Solo quedan Semaglutide y Zepbound, la app concluye que su tratamiento "no está disponible" y muestra el modal.

Claudio está diciendo de importar esos MedIds como `Salesforce::Medication` pero no sé si deba hacer eso.