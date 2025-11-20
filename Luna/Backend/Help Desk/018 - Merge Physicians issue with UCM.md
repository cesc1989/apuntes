# 018 - Merge Physicians breaks because of UserCommunicationMethod dependency

Etiquetas: #luna_help_desk 

Caso EDG-3031

Ken trató de mezclar dos physicians pero no se completó y dio error 500. El error es porque el physician a borrar tiene una relación entre UCM y la tabla `outbound_transmission_attempts` que impide que se complete la transacción.

## Contexto

Cuando se creó la tabla `outbound_transmission_attempts` se relacionó con UCM mediante esta restricción:
```ruby
t.belongs_to :user_communication_method,
	null: false,
	foreign_key: { on_delete: :restrict },
	type: :uuid,
	index: false
```

Migración: `db/migrate/20220928195736_create_outbound_transmission_attempts.rb`

Sin embargo, en un PR muy posterior, se borró toda funcionalidad alrededor de esta tabla.

Ver PR: https://github.com/lunacare/backend/pull/11666

Se borraron archivos pero nunca se borró la tabla ni la restricción.

### Merge Physicians

La UI de esta funcionalidad está en https://luxe.alpha.getluna.com/admin/assist

El código que lo soporta está en `app/services/physician_merge_service.rb` y el controlador está en `app/admin/actions/assist.rb`.

# Soluciones

Claudio sugirió hacer una actualización en la función que hace el merge de los physicians. Yo creo que es mejor borrar la asociación.

Sin embargo, la solución de Claude está bien porque no sé que datos históricos se puedan dañar por borrar la restricción.

Haré los cambios como sugiere Claude y en el PR preguntaré si mejor borro la restricción.

## Reasociar el Outbound Transmission Attempt

Hay que moverlo al physician que se va a mantener.

# Pruebas

## En Local

Target:
```
8e45f50f-785c-4916-ac14-d93437e707d0
```

- HubSpot ID: 647001
- NPI original: 0000066143
- NPI en hubspot: 2023021311

> [!Tip]
> Para poder probar en local el NPI del contacto en HubSpot y del Physician en la BD deben ser el mismo.

> [!Tip]
> El NPI debe ser de 10 dígitos. Y debe ser único.

> [!Tip]
> Si hay error de HubSpot ID, confirma que no haya más physicians con un mismo HubSpot ID. De pronto no lo hay pero por correo pueden ser el mismo contacto.
>
> Ejemplo, el email de esta prueba tiene dos physicians en la BD.

- Email: `yuly.murillo+ph-20230213223548@koombea.com`

Source:
```
8d36fa4c-4f9d-4b3c-b88c-3107144df8c9
```