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