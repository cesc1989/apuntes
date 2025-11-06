# 012 - Therapist Boosted está contando Waitlist self-selected

Etiquetas: #luna_help_desk 

Caso EDG-2899

> [!Tip]
> El formulario para crear un boost carga MUY lento. Podría ser algo para mejorar para open dev fridays.
> 
> Carga muy lento porque la lista de selección carga a todos los therapists.

Reporte:

> Boosted Therapist feature counting self selected Waitlist patients from therapists as organic bookings

# Contexto

Cuando un therapist está con pocos pacientes para atender puede activar una opción para tomar los pacientes de "Waitlist". Además, cuando están cortos de pacientes desde Luna está la opción de darles un "boost" de manera temporal para darles prioridad en la recepción de pacientes.

Cuando se "boostea" se defina una cantidad de pacientes que quieren ver antes de quitar el "boost".

La información sobre el estado de Waitlist y la disponibilidad del paciente se muestra en una caja en la parte superior derecha del perfil.

![[patient.waitlist.status.png]]

Nota como hay dos botones:

- Edit Waitlist Availability
- Manage Waitlist Status

Es porque ambos manipulan modelos diferentes.

## Waitlist status

Se define en el modelo `WaitlistEntry`. Este es el modelo donde se define el estado del paciente sobre si está o no en Waitlist.

Ver el enum status en [[Edge Enums que Importan 🔑#WaitlistEntry]]

## Availabilities

Se define en el modelo `Availability`. Aquí se gestiona la lista de horarios en el que el paciente en Waitlist puede recibir visitas (appointments).

# Problema

Cuando el therapist es "boosted" y además también activó la opción de tomar pacientes del "Waitlist" el sistema está contando ese "Waitlist" como si Luna lo hubiera agendado. Lo cual es incorrecto.

En el reporte dice:
> Waitlist patient selections should not apply to this count.

# Replicar el Error 🐞

## Therapist Boost

Hay que crear un "Therapist Boost" para cualquier therapist. Esos se crean yendo a http://localhost:3000/admin/therapist_boosts/new

> [!Note]
> La página es MUY lenta para cargar. Recomiendo usar el 1er therapist que cargue.

## Paciente en Waitlist

Necesito encontrar un paciente que esté con Waitlist status activo y que tenga availabilities. Puedo usar esta query:
```sql
SELECT
 pat.id,
 we.status
FROM patients pat
join accounts acc on acc.id = pat.account_id
join availabilities a on a.account_id = acc.id
join waitlist_entries we on we.patient_id = pat.id
where we.status = 0
group by pat.id, we.status
;
```

## Seleccionar Paciente en Waitlist como Therapist

Finalmente, hay que hacer como si desde la app móvil el therapist escogiera al paciente que está en Waitlist. Esto lo podemos hacer mediante GQL.

Ver [[Auth de GraphQL con Postman]]

Se puede emular esta selección con esta mutación:
```json
mutation {
  addAppointment(
    input: {
      userId: "9dcdac51-8f52-46a7-8a59-525274dfa69d"
      startTime: "2025-11-01T14:00:00-07:00"
      publishNow: true
    }
  ) {
    appointment {
      id
      state
    }
    userErrors {
      message
      field
    }
  }
}
```

> [!Important]
> Los tiempos de cada availability deben ser de 45 minutos. Eso es el tiempo estándar de cada appointment en Luna.

---

Con todo lo anterior creado se puede comprobar el error al revisar en algunas partes en el perfil del Therapist.

- En la sección "internal notes" aparecerá este mensaje:
> Boosted! 🚀 1 of 9 target new cases fulfilled. (Boosted on 10/29/2025 Wed, ends 11/15/2025 Sat)

Lo cual está mal porque sale es por el paciente tomado de Waitlist.

- En la lista de Appointments, el que se creó del paciente en Waitlist tendrá el emoji 🚀 en la columna "Boost?"

Esto también es erróneo por la razón anterior.

# Solución

La solución se da por aplicar un filtro a la consulta en el modelo `TherapistBoost` que calcula la cantidad de casos boosteados.

Así se ve sin el fix:
```ruby
def cases
	Episode.where(
		therapist
			.appointments
			.where("episodes.id = appointments.episode_id")
			.without_clerical
			.where(visit_type: "initial")
			.where(created_at: effective_from..boost_end)
			.where(created_by_type: "AdminUser")
			.where(canceled_by_type: [nil, "Therapist", "Patient"])
			.select(1)
			.arel.exists
	)
end
```

Esta función se usa en varias partes de la UI (perfil del Therapist) y también en otros cálculos de este modelo. Como en las funciones:
```ruby
def target_cases_fulfilled?
	cases.count >= cases_target_delta
end

def target_cases_remaining
	[cases_target_delta - cases.count, 0].max
end
```

