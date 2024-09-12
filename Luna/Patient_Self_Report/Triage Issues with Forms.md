# Triage de Issues en Forms: Intake and Progress

Fecha: 12 de Septiembre, 2024.
Reportes:
- FOR-273: (P2) Therapist app incorrectly links incomplete progress form
- FOR-275: (P2) Intake forms incorrectly reverted to incomplete status
- FOR-197: (P2) Luxe and Therapist App missing link to Progress Form in Luna Therapist app
	- Related:
		- FOR-299: (P3) Therapist app incorrectly states form is due
		- FOR-300: (P2) Progress form freezing therapist app due to link issue

# Datos para FOR-273

> P2: Therapist app incorrectly links incomplete progress form

En el [Summary Page](https://forms.getluna.com/v3/patients/430ea2a6-a35c-4ca1-bbe3-71bac0a5a98f/summary/b4d424b0-3e6e-40d0-8ae1-2f8dd4001389) del Intake se puede ver que:
- Intake está completo
	- ID: `1edf6996-4e87-4a5f-bd20-737af8a28186`
- Hay dos progress forms:
	- PF2:
		- Sin completar
		- Creado 20 de Agosto
		- ID: `ad259bc8-f640-4b6d-aca4-638dcb16b44d`
	- PF1:
		- Sin completar
		- Creado 12 de Septiembre
		- ID: `7b2d47b0-fff6-48a5-b4a9-38274d6b4f22`

El asunto es que en la app, el estado del Intake Form se indica como "incompleto" pero en realidad está completo (se puede ver en el summary) y lo que debería decir es que:
- El PF1 está incompleto
- Enlazar al PF1

# Datos para FOR-275

> P2: Intake forms incorrectly reverted to incomplete status
> 
> Relacionada con FOR-273

En el [Summary Page](https://forms.getluna.com/patients/1c9b1f3d-21ca-44f5-925a-72b00cfe4bc7/summary/17da8388-2bfb-4320-9c9d-7b806d47f45f) se ve que el Intake Form está completo.

Sin embargo, noto varias cosas:

- En la grabación, el Intake Form que abre, y aparece como _incompleto_, su ID empieza por: `383c9074`
- Sin embargo, el enlace al Summary Page carga otro Intake Form cuyo ID empieza por: `d42cd542`
	- Este Intake Form sí está completo.

¿Qué hay que hacer aquí? Parece que hay algún tipo de enredo con los diferentes care plans que puede tener un mismo paciente.

# Datos para FOR-197

> P2: Luxe and Therapist App missing link to Progress Form in Luna Therapist app
> 
> Está relacionada con FOR-299 y FOR-300


## Datos para FOR-299

> P3: Therapist app incorrectly states intake form is due
>
> The therapist's app says Intake form Incomplete, but luxe confirms that it is complete.

En el [Summary Page](https://forms.getluna.com/v3/patients/2639bcea-7041-498a-850e-813e6a1a1c95/summary/1f93dfbc-c64a-4250-83aa-68abe1866f50) podemos ver varias cosas:
- El Intake Form está completo
	- ID: `ad636146-ea40-4930-85d3-a818a857b603`
- Tiene dos progress forms
	- PF2
		- Está completo
		- ID: `04271227-4168-44a1-a1f6-c0f206340b3b`
	- PF1
		- Está incompleto
		- ID: `4c4aa9ee-d0da-4dca-bb11-a09f76783751`
- En la grabación se ve como el problema es en la Therapist App

Parece que todo está relacionado al API V2 de Forms y el endpoint que da los estados de cada form.

## Datos para FOR-300

> P2: Progress Form freezing Therapist app due to Luxe link issue

En la grabación se ven varias cosas:
- El therapist dice que:
	- llenaron 3 veces el progress form pero siempre se _freezeaba_ antes de hacer submit
- En la sección **Latest Forms** del perfil del paciente carga un Intake Form como *complete* y no carga Progress Form.
- El paciente, en el care plan activo, tiene **dos Intake Forms seguidos** y un Progress Form.
	- Nota que antes el paciente tuvo un care plan en estado _auto discharged_

¿Qué falló aquí? Necesito los enlaces de los Intake Forms y del Progress.

Intake 2024-08-16 [view](https://forms.getluna.com/patients/0cde0395-f774-43c3-9e10-f6c8c565164b/summary/09c79dbb-fb63-426d-a303-76fd47730abe)

Intake 2024-08-19 [view](https://forms.getluna.com/patients/0cde0395-f774-43c3-9e10-f6c8c565164b/summary/ed189d1e-6f6f-483d-9340-5ed8922471c3)

Progress 2024-08-31 [view](https://forms.getluna.com/patients/0cde0395-f774-43c3-9e10-f6c8c565164b/summary/bfa9d6e4-4813-4ad6-9dcf-5f84d10625b7)