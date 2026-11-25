# OM Ciclo 60

> [!Info]
> Del Jueves 19 de Noviembre hasta el Miércoles 2 de Diciembre.

## Caso OM-12842 - Reroute to Beluga de CX en Salesforce 🟢

Etiquetas: #om_reroute_to_beluga #om_reroute_to_beluga_salesforce

Ya sabemos cómo hacer el reroute cuando el CX está en Ontraport. Este caso es cuando está en Salesforce.

### Verificaciones

Hay que encontrar el Member Period actual.

- Debe estar en estado `VisitCreated`
- CE en Scheduled con source system CareValidate.
- Recomendación en RecommendationMade con providers "CareValidate".
- CV request en waiting_for_prescription.

Hay que correr, en dry-run, el copier con Beluga para confirmar que MedPicker encuentra un medicamento con prescriber Beluga. Si truena con NoEquivalentProducts, Beluga no surte ese producto y no hay que seguir.

### Comandos

Buscar el MP:
```ruby
acct = Account.find("<ACCOUNT_UUID>").salesforce_account
acct.member_periods.each do |mp|
  pp({
    mp_omid: mp.omid,
    status: mp.status,
    source_created: mp.source_system_created_at,
    ces: mp.clinical_encounters.map { "#{it.omid} #{it.status} #{it.source_system}" },
    recs: mp.med_picker_recommendations.map { "#{it.omid} #{it.status}" }
  })
end; nil
```

Verificar el CE, la recomendación y el request de CV:
```ruby
ce = Salesforce::ClinicalEncounter.find_by!(omid: "<CE_OMID>")
rec = ce.med_picker_recommendation
cv = CareValidate::Request.find_by(clinical_encounter_id: ce.omid)

pp({
  ce_status: ce.status,
  mp_status: ce.member_period.status,
  rec_status: rec.status,
  rec_providers: rec.providers.map(&:name),
  products: rec.selected_products.map { "#{it.product&.name} -> #{it.recommended_medication&.description} [#{it.recommended_medication&.prescriber&.name}]" },
  cv_id: cv&.id,
  cv_state: cv&.state,
  cv_case_nk: cv&.case_nk
})
```

Dry-run con Beluga:
```ruby
rec = Salesforce::ClinicalEncounter.find_by!(omid: "<CE_OMID>").med_picker_recommendation
beluga = Salesforce::BusinessAccount.partner_medical_service_organizations.find_sole_by(name: "Beluga")

Salesforce::Base.transaction do
  copier = Salesforce::MedPickerRecommendationCopier.new(rec)
  copier.providers = [beluga]
  new_rec = copier.copy
  pp({
    changes: copier.changes.keys,
    new_products: new_rec.selected_products.map { "#{it.product&.name} -> #{it.recommended_medication&.description} [#{it.recommended_medication&.prescriber&.name}]" }
  })
  
  raise ActiveRecord::Rollback
end
```

> [!Note]
> Si lo anterior es correcto, viene el reroute.

Reroute: el request de CV se cancela fuera de la transacción porque está en otra base de datos.
```ruby
ce = Salesforce::ClinicalEncounter.find_by!(omid: "<CE_OMID>")
mp = ce.member_period
cv = CareValidate::Request.find("<CV_REQUEST_ID>")

raise "MP: #{mp.status}" unless mp.status == "VisitCreated"
raise "CE: #{ce.status}" unless ce.status == "Scheduled"
raise "CV: #{cv.state}" unless cv.state == "waiting_for_prescription"

Salesforce::Base.transaction do
  mp.update!(status: "ReadyToCreateVisit")
  CareValidate::RerouteToBeluga.call(clinical_encounter: ce)
end

cv.cancel!
cv.update!(canceled_at: Time.current, notes: "Rerouted to Beluga (SYNC state) - <TICKET>")

pp({
  mp_status: mp.reload.status,
  cv_state: cv.reload.state,
  ces: mp.clinical_encounters.reload.map { "#{it.omid} #{it.status} #{it.source_system}" },
  recs: mp.med_picker_recommendations.reload.map { "#{it.omid} #{it.status} #{it.providers.map(&:name)}" }
})
```

> [!Important]
> Qué deberías ver:
> 
> - el CE viejo en `Cancelled`
> - un CE nuevo en `Pending` con `BelugaHealth`
> - una recomendación nueva con `["Beluga"]`
> - y el MP en `ReadyToCreateVisit`.

