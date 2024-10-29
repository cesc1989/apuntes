# Pruebas y otras cosas

# Prueba de CompletedForm falla por el orden de los atributos

La prueba en `spec/lib/webhooks/completed_form.rb` falla porque la clase devuelve los atributos en un orden distinto a como se generan en la prueba:
```ruby
# En la clase
'{"form":{"progress_type":"onboarding","injury_name":"InjuryName1","pain_scale":5,"uuid":"b254261a-1769-42ce-84f0-8a14a54eb3fb","completed_at":"2018-10-14T11:00:00.000Z","id":1},"patient":{"id":1,"full_name":"FirstName1 LastName1","internal_id":"58e194a8-9f6c-4de5-a0d4-266deec83f30"}}'

# En la prueba
"{\"form\":{\"id\":1,\"progress_type\":\"onboarding\",\"injury_name\":\"InjuryName1\",\"completed_at\":\"2018-10-14T11:00:00.000Z\",\"uuid\":\"b254261a-1769-42ce-84f0-8a14a54eb3fb\",\"pain_scale\":5},\"patient\":{\"id\":1,\"full_name\":\"FirstName1 LastName1\",\"internal_id\":\"58e194a8-9f6c-4de5-a0d4-266deec83f30\"}}",
```

Para arreglarlo tuve que cambiar el orden del payload en la prueba.

De esto originalmente:
```ruby
let(:payload) do
	{
		"form": {
			"id": form.id,
			"progress_type": form.progress_type,
			"injury_name": form.injury_name,
			"completed_at": form.completed_at,
			"uuid": form.uuid,
			"pain_scale": form.pain_scale
		},
		"patient": {
			"id": patient.id,
			"full_name": patient.full_name,
			"internal_id": patient.internal_id
		}
	}
end
```

A esto modificado:
```ruby
let(:payload) do
	{
		"form": {
			"progress_type": form.progress_type,
			"injury_name": form.injury_name,
			"pain_scale": form.pain_scale,
			"uuid": form.uuid,
			"completed_at": form.completed_at,
			"id": form.id
		},
		"patient": {
			"id": patient.id,
			"full_name": patient.full_name,
			"internal_id": patient.internal_id
		}
	}
end
```

¿Por qué pasa?