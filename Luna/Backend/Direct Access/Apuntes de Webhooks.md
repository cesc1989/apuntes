# Apuntes de Webhook de Therapist

Datos básicos:

Controlador: `app/controllers/therapist_controller.rb`

Rutas:
```ruby
# incoming hubspot webhooks
post "patient/merge_patients"
post "patient/sync_merged_contacts"
post "physician/sync"
post "physician/sync_merged_contacts"
post "therapist/sync"
post "therapist/sync_merged_contacts"
```

Pruebas: `spec/requests/therapist_controller_spec.rb`