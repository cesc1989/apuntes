# 011 - Sync Care Plan Latest Auth Effective Until to HubSpot

Etiquetas: #luna_help_desk 

Caso EDG-1854

Pidieron sincronizar la fecha que aparece en el recuadro "Latest Auth Effective Until" que se muestra en la sección Care Plan del perfil de un paciente.

## Contexto

Dicha fecha se determina en `app/admin/care_plans.rb:385` así:
```ruby
row "Latest Auth Effective Until" do |care_plan|
	latest_date = care_plan&.latest_authorized_visit_date

	next if latest_date.blank?

	I18n.l(latest_date)
end
```

Y así luce el método `latest_authorized_visit_date` en el modelo Episode
```ruby
def latest_authorized_visit_date
	cache_key = "#{cache_key_with_version}/latest_authorized_visit_date"

	Rails.cache.fetch(cache_key, expires_in: 5.minutes, skip_nil: true) do
		PayerAuthorization.latest(authorizations.to_a.select(&:granted?))&.effective_until
	end
end
```



## Solución

Hay que completar este sync desde dos frentes:

- Clase `Hubspot::SyncPatientService`
	- Para cubrir:
		- Manual sync desde Luxe
		- Sync general
- ~~Callbacks en `Episode` cuando se cree o actualice el care plan~~
- Callback en `PayerAuthorization`

### Clase `Hubspot::SyncPatientService`

Se usa para hacer un update al completo del contacto en HubSpot. Se invoca unicamente desde `Hubspot::SyncPatientWorker`. Este worker es encolado desde diferentes partes del sistema:

Le pedí a Claudio encontrar los lugares desde donde se encola al worker. Esto encontró:

**Patient Creation**
  - Trigger: `app/models/patient.rb:192`
  - When: After a patient is created (via after_create_commit)
  - Condition: Patient is not in draft status
  
  ```ruby
  after_create_commit :sync_hubspot, unless: :draft?
  ```

**Patient Onboarding Finalization**
- Trigger: `app/grimoire/omni/actions/finalize_patient_onboarding.rb:61`
- When: When patient onboarding is completed in Omni

**Case Creation Finalization**
- Trigger: `app/grimoire/omni/actions/finalize_case_creation.rb:57`
- When: When a new case (care plan) is finalized in Omni

**Patient Merge Operations**
- Trigger: `app/services/patient_merge_manager.rb:31`
- When: After patients are merged

Reward Updates
- Trigger: `app/models/reward.rb:37`
- When: After a reward is updated (delayed by 10 seconds)

**Tip Creation**
- Trigger: `app/models/tip.rb:78`
- When: After a tip is created (delayed by 10 seconds)

**Batch Sync Operations**
- Triggered by:
	- `HubspotSyncContactsService` - Bulk contact syncing
	- `HubspotSyncMultiplePatientsWorker` - Multiple patient syncing
	- `HubspotSyncPatientsWithUpdatedEpisodesWorker` - Syncs patients whose episodes were updated in the last 7 days

**Manual Admin Sync**
- Trigger: `app/admin/customers/patients.rb:362`
- When: Admin manually triggers sync from patient admin page

### Callback en PayerAuthorization

Para solo lanzar un sync a HubSpot cuando se cree, actualice o borre un registro de `payer_authorizations` el callback lo moví a este modelo.

### Callbacks en Episode

> [!Warning]
> Esta alternativa no la implementé. Dado a que Ryan pidió hacer el sync cuando el valor de `latest_authorized_visit_date` haya cambiado.
>
> Dado a que eso es calculado no es posible sacar el anterior y el posterior porque el sync se da desde un callback `after_commit`.
>
> La alternativa es hacer el sync desde el modelo `PayerAuthorization`.

Para hacer syncs a HubSpot desde el care plan el sistema se vale de varios callbacks cuando se crea o actualiza un care plan. Estas funciones usan el worker `Hubspot::UpdateContactPropertiesWorker` para actualizar propiedades puntuales de cada Contacto.

Ejemplo:
```ruby
after_commit :sync_hubspot_active_status, on: [:create, :update]

# (...)

def sync_hubspot_active_status
	return if draft?
	return unless most_recent_care_plan?

	Hubspot::UpdateContactPropertiesWorker.perform_async(
		patient_id,
		{
			active_care_plan: active?,
			care_plan_scheduling_enabled: discharged? ? nil : scheduling_enabled
		}.as_json
	)
end
```

