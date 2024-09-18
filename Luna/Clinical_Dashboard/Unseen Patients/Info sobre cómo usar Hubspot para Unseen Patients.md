# Info sobre cómo usar Hubspot para tener datos para Unseen Patients: Physicians o PBLs

Como toda esta información está en Hubspot no hay forma tan sencilla de tener datos para Alpha. Toca ir a los contactos y hacer las asociaciones por etiquetas o por propiedades.

En Hubspot, me valgo de varias vistas para entender qué datos hay disponibles que contactos puedo enlazar a Physicians o PBLs.

# Para Physicians 👩🏻‍⚕️

Tengo una vista que se llama "Alpha CD Physicians". En esta se listan tres physicians:

- TestA TestSalya - 19976815111
- Hugh ThePhysician - 265101
- Lina Lew E - 9101

Para poder mostrar pacientes en los widgets de Unseen Patients necesito buscar pacientes de prueba. Estos son generados por las corridas de automation.

> [!INFO]
> Hay una enorme cantidad de pacientes de prueba en la vista "Addable to EVE".

> [!WARNING]
> Si quiero que aparezcan en la tabla de Unseen Patients, tengo que enlazar los que fueron creados en una fecha *mayor* a 90 días y *menor* de 2 días.

Para asignar pacientes a estos physicians debo ir al perfil del paciente a enlazar y en la barra derecha ubicar el menú "Contacts" y seguir así:

1. Clic en "+ Add"
2. Buscar el physician por el correo o nombre
	1. Seleccionar y clicar en Next
3. Clic en "+ add association label"
4. En la caja de búsqueda elegir "Referring Physician"
5. Clic en Save

Debería verse así:
![[associate.physician.to.patient.png]]

También se puede al revés. Ir al perfil del physician y buscar los pacientes. Sin embargo en ese modo es más difícil ubicar los pacientes que cumplan con el rango de fechas. Si es solo para hacer cantidad, sí es más breve porque se pueden seleccionar varios a la vez.


# Para PBLs 🏥

Para Providers la cosa cambia. Para más contexto ver:

- [[Unseen_Patients_for_PBLs_-_Notes_and_Findings]]
- [[Hubspot Search API]]

> [!INFO]
> Hay una enorme cantidad de pacientes de prueba en la vista "Addable to EVE".

Para PBLs es un poco más fácil. En Alpha suelo usar los Practices:

- Evergreen Health - EVE
- Luna A Proud Partner of UCI - UCI

Para asignar pacientes a EVE, por ejemplo, voy a la lista "Addable to EVE" y elijo pacientes que fueron creados en una fecha mayor a 90 días.

Aquí es más fácil porque puedo asignar en bultos usando la UI de Hubspot. Se seleccionan los pacientes usando el checkbox a un lado del nombre y se modifican las propiedades:

- Powered by Luna Partner Code para que sea igual a EVE
- First Touch Lead Source para que sea Powered by Luna

Como se ve en la imagen:
![[associate.patients.to.pbls.png]]

Y así aparecerán en los widgets de Unseen Patients cuando se filtra por Practice o Clinic.