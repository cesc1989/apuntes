# OM Ciclo 52

> [!Info]
> Del Jueves 30 de Julio al Miércoles 12 de Agosto.

## Caso OM-10287 - Unable to recommend script 🟢

Etiquetas: #om_unable_to_recommend_script 

> [!Important]
> Lo primero a hacer en estos casos es revisar las sensitivities marcadas en Ontraport. Si están todas, eso puede ser el problema ya que no se vende el medicamento puro.

Al revisar se ve esto:
![[om_10287.png]]

Jaime me explicó que no se vende medicamento sin ningún aditivo. Los aditivos están presentes de esta forma:

- con B6
- con B12
- con B3 y B5

El cliente marcó todas así que no se puede recomendar ninguna opción. La resolución de estos casos es hacer un reembolso y crear un nuevo Script. No se pueden modificar esos valores.

### Actualizaciones ℹ️

Estos casos de sensitivities se pueden resolver de dos formas:

1. Hacer un reembolso y crear un nuevo script
2. Reiniciar todo el proceso desde el HealthScreening

Fabian explica que si el cliente sí quiere ordenar, entonces lo que toca hacer es el paso 2. Devolverlo al HealthScreening. ¿Por qué? Porque si se hace refund + nuevo Script entonces el cliente no será capaz de cambiar la selección de los Sensitivities en el nuevo Check In.

> [!Warning]
> Lo segundo no hay nada confirmado aún. Pendiente de Thomas de confirmar a Jay. Mientras tanto sigue haciendo lo mismo. Indica el problema y mandale el problema a CS.

## Casos de blank screen cuando el CX iba a pagar 🟢

Etiquetas: #om_http_error_400 

Casos:

- OM-10645
- OM-10657

Clientes que habían completado el Check In y en la página de pago se encontraban con una página en blanco. Cuando impersoné llegué a un error HTTP 400:
```
HTTP ERROR 400
```

Pedí a claudio ayuda y resulta que ese error, al intentar ir a la página:
```
https://patient.orderlymeds.com/logins/external_auth?external_auth_id=01KYWV2M894B4PA7178MDMA2WN
```

si el Account no tiene el campo `work_os_nk` se devuelve error 400. Eso se ve en `app/controllers/patient/logins_controller.rb`:
```ruby
def external_auth
	external_auth_id = params[:external_auth_id]
	return head(:bad_request) if external_auth_id.blank?
	return head(:bad_request) if current_account.workos_user_nk.blank?

	workos_user = WorkOS.client.user_management.get_user(id: current_account.workos_user_nk)

	redirect_to response.redirect_uri, allow_other_host: true
end
```

El parámetro sí estaba en la URL así que el error era el campo. Cuando revisé ambas cuentas ninguna tenía el valor correspondiente.

El fix fue correr el mismo comando para el problema de "Oops Error".

## Caso OM-10722 - Oops Error al ir al Checkout 🟢

Etiquetas: #om_mp_missing_medication_reference

CX con Check In completo, valor correcto en `workos_user_nk`. Al navegar al Checkout da el error de "Oops error".

La URL a la que se navega es:
```
https://patient.orderlymeds.com/checkin/purchases/new
```

Encontré en Rollbar el error usando el ID del Account:
```
ActiveRecord::AssociationTypeMismatch: Salesforce::BusinessAccount(#44352) expected, got nil which is an instance of NilClass(#8)
```

Claudio me dio un script para verificar en consola y dio lo mismo:
```
app=orderlymeds environment=production at=med_picker_recommendation_for_beluga module=Patient::Checkin::MedPickerRecommendationBuilder account_id=019ee70c-f6e6-731f-9f65-d7ead314fc7c message='Requesting MedPicker recommendation for Beluga Patient Preferences'

ActiveRecord::AssociationTypeMismatch: Salesforce::BusinessAccount(#42720) expected, got nil which is an instance of NilClass(#8)
```

### Problema: Missing Medication Reference en MP migrado ℹ️

Pasa que en el más reciente MP generado por la migración de Ontraport a Salesforce el Medication Request dice en el campo "Medication": "Missing Medication Reference".

Esto sale cuando reviso dicho Medication Request:
```ruby
sfa.medication_requests.last.medication__omid__c
=> "NULL_MEDICATION_OMID"
```

También se puede ver en Case Overview. Así se veía cuando estaba causando el error:
![[OM_10722.01.png]]

Y así se ve una vez apuntado al MedId correcto (en este caso apuntamos al mismo del Medication Request anterior):
![[OM_10722.02.png]]

#### Diagnóstico y Solución

Con esto se hace el diagnóstico:
```ruby
email = "cx@example.com"
sfa = Account.find_by!(email: email).salesforce_account

sfa.medication_requests.active_or_completed.for_glp1_products.order(prescribed_date: :desc).each_with_index do |mr, i|
  m = mr.medication
  puts "[#{i}] #{mr.omid} #{mr.prescribed_date} status=#{mr.status} sfid=#{mr.sfid}"
  puts "    med=#{m&.omid.inspect} #{m&.name.inspect} status=#{m&.status}"
  puts "    prescriber=#{m&.prescriber&.name.inspect}   <== nil = roto"
end

# [0] con med=NULL_MEDICATION_OMID y prescriber=nil confirma el caso. El [1] (o el primero con prescriber) es la medicación real a usar.
```

Así fue el ejemplo del caso:
```
[0] medication_request 019ee70d-007c-7718-befa-f86233e66f9e prescribed_date=2026-06-06 11:51:16 UTC status=Active
    medication:     "NULL_MEDICATION_OMID" "Missing Medication Reference"
    prescriber FK:  nil
    prescriber:     nil   <== nil aqui = ESTE revienta
    cuenta cruda:   FK vacio en la fila de medication

[1] medication_request 019ee70c-ff4d-7500-8edb-1a4e0adda118 prescribed_date=2026-02-13 03:14:00 UTC status=Completed
    medication:     "019ee70c-ff10-77a4-9ac5-4489a5cb7dda" "Semaglutide 4.5mg/ml (6.75MG) 6.75MG/1.5ML"
    prescriber FK:  "0198c33c-5d09-7db3-9c8b-9bcab06d3749"
    prescriber:     "CareValidate"   <== nil aqui = ESTE revienta
    cuenta cruda:   name="CareValidate" recordtypeid="012Pm000007800PIAQ" ispersonaccount=false isdeleted=false hc_err=nil
    record type OK? true
```

Luego hay que validar antes de actualizar:
```ruby
mr  = Salesforce::MedicationRequest.find("<omid del [0]>")
med = Salesforce::Medication.find_by!(omid: "<omid del medication del [1]>")
prod = med.product

puts "prescriber: #{med.prescriber&.name.inspect}"                                            # no nil
puts "form ok?      #{Salesforce::Commerce::Product::FORMS.include?(prod&.form)}"             # true
puts "treatment ok? #{Salesforce::Commerce::Product::TREATMENTS.include?(prod&.treatment)}"   # true
puts "treatment_form: #{prod&.treatment_form.inspect}"                                        # no nil
puts "titration_final_rx_strength: #{med.titration_final_rx_strength.inspect}"                # anotar

# No sigas si alguno falla. med.status == "Inactive" es normal y no bloquea.
```

Se corrige:
```ruby
mr.medication = med
mr.save!
mr.reload
puts "ahora: #{mr.medication.omid} #{mr.medication.name.inspect} prescriber=#{mr.medication.prescriber&.name.inspect}"
```

Verificación final:
```ruby
prev = sfa.medication_requests.active_or_completed.for_glp1_products.order(prescribed_date: :desc).first
puts "elige: #{prev.medication.name.inspect} prescriber=#{prev.medication.prescriber&.name.inspect}"

Patient::Checkin::MedPickerRecommendationBuilder.call(
  member_period: sfa.latest_member_period,
  previous_medication: prev.medication
)
```

Y que al hacer el impersonate e ir a la página no explote sino que cargue el Checkout.

## Caso OM-10556 Bundle manually rejected 🟢

Etiquetas: #om_bundle_manually_rejected #om_bundle_issue 

CX en Ontraport con Beluga como prescriber. La orden no llegaba a la farmacia porque el bundle decía "manually_rejected". Había que hacer un resubmit al Script.

Fui a Ontraport e hice el resubmit normal. Vi que los registros en la pestaña "Incoming Webhooks" desapareció mientras se procesaba. Igual que en "RxWritten Bundles" y "Beluga Submissions".

Cuando el proceso se completó el registro en "Incoming Webhooks" quedó así:
- Source: beluga_health
- Event: prescription_written
- State: delivered

Y el de "RxWritten Bundles" así:
- State: automatically_approved

Finalmente, el de "Beluga Submissions" decía "Sent to Pharmacy" y la fecha de hoy. En "Orders" los campos Written, Sent, Received estaban todos con el chulo verde.