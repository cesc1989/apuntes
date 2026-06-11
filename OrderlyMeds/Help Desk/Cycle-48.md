# Ciclo 48

## Caso OM-9224 - Resend Invoice 🟡

Dicen esto:
> Customer request an invoice copy that clearly says 'invoice copy'. Details should be like this: (she's kinda irate)
> • Name of person who received the item or service  
> • Description of the item or service  
> • Date of service  
> • Retailer or service provider name  
> • Amount you paid

Al ver el perfil en OM se ve que es un cliente en Salesforce. Por esto, Jaime me explicó que hay un botón para reenviar un invoice al presionar "Create New Case" y elegir el Case Type.

### Sobre la fecha del Invoice

En el ticket, Ronalie comentó:
> please send the 4/27 invoice

Cuando voy a ver los MPs en Salesforce no veo ninguna fecha que corresponda a la indicada.
![[om_9224.01.png]]

Al preguntar a Jaime me explica que es la fecha del Order. Se ve en el detalle del Member Period en Salesforce en la sección "Order Summaries".
![[om_9224.02.png]]

### Dudas sobre el Invoice reenviado

Lo que pide el cliente es muy específico y no me queda claro si el mailer que indicó Jaime en el repo correspondo a esa acción que efectúa Salesforce.

El mailer que dice Jaime está en la ruta `app/views/patient/order_summary_mailer/send_invoice.html.erb`.

No me suena porque el mailer tiene son estos datos:

- first_name
- last_name
- @order.order_reference_number
- @order.effective_date

Esto:
```ruby
<% @order.order_items.each do |item| %>
	<li>Product & Plan: <%= item.product.name %></li>
	<li>Preferred Treatment: <%= item.product.humanize[:treatment] %></li>
<% end %>
```

- Subtotales
- Shipping Address

No tiene nada de lo que piden en particular.

## Caso OM-9214 - Reasignado a Jaime 🔵

> [!Info]
> La clave de este caso es que Beluga prescribió una medicina que no era la correcta o faltó agregar una recomendación.
> 
> (ver recomendaciones del MP en Salesforce en la sección "Med Picker Recommendation").
>
> Estos casos (hasta nuevo aviso) hay que escalar a Amber de Beluga para preguntar sobre dicho bundle. Se escala con el _master id_ del bundle.

Dice:
> The customer encountered a bundle issue with the order. The bundle has been rejected, and an error occurred while attempting to resubmit the order.

> [!Important]
> Cuando hagan referencia a un problema con el _bundle_ están haciendo referencia a algo que pasó con Beluga.
>
> Ver [[OrderlyMeds Glossary]] para ver qué es Beluga.

En este caso, Jaime fue al perfil del CX en Success, copió el "Latest master id" y navegó a la sección "FlaggedRxWrittenBundles". En esa página hizo una búsqueda con el master id copiado y de los resultados elegir el más reciente.

> [!Note]
> Se puede constatar que sea el "latest master id" desde Salesforce yendo al MP más reciente y revisar la columna "Source System ID" en la sección "Clinical Encounters".

Al entrar al detalle del `rx_written_bundles` se ven varias secciones. Para este caso, en la sección "All" había esto:
![[om_9214.01.png]]

Donde, al detallar, se ve que la prescripción no coincide o no incluye la recomendación hecha desde el Med Picker.