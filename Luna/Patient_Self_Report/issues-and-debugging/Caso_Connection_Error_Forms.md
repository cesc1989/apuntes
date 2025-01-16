# Caso: Connection Error Forms ✅
Form Link https://forms.getluna.com/patients/cb2cd416-0341-432c-8775-ca6605ec2d16/forms/84524eed-26ef-41c9-b79b-b68ed8cc1d96

El formulario hace estas peticiones:

    /patients/INTERNAL_ID/forms/UUID
    
    https://forms-api.getluna.com/patients/cb2cd416-0341-432c-8775-ca6605ec2d16/forms/84524eed-26ef-41c9-b79b-b68ed8cc1d96
    
    1. aggravating_activities:
      0: {name: "", ability: 0}
      1: {name: "", ability: 0}
      2: {name: "", ability: 0}
      3: {name: "", ability: 0}
      4: {name: "", ability: 0}
      5: {name: "", ability: 0}
      6: {name: "rising from sitting position", ability: 4}
      7: {name: "walking", ability: 5}
      8: {name: "balance", ability: 4}
    2. created_at: "2020-07-06T19:56:41.083Z"
    3. exclude_medical_information: false
    4. form_type_id: 5
    5. id: 11258
    6. injury_name: "Other"
    7. onboarding_completed: true
    8. patient: {full_name: "Inez Schneider", first_name: "Inez", last_name: "Schneider", gender: "female",…}
    9. progress_type: "ongoing"
    10. terms_and_conditions: true
    11. type_name: "psfs"
    12. uuid: "84524eed-26ef-41c9-b79b-b
    

Y esta

    /forms/UUID/previous_progress_forms
    
    https://forms-api.getluna.com/forms/84524eed-26ef-41c9-b79b-b68ed8cc1d96/previous_progress_forms
    
    aggravating_activities_attributes:
        0: {name: "", ability: 0}
        1: {name: "", ability: 0}
        2: {name: "", ability: 0}
        3: {name: "", ability: 0}
        4: {name: "", ability: 0}
        5: {name: "", ability: 0}
        6: {name: "rising from sitting position", ability: 4}
        7: {name: "walking", ability: 5}
        8: {name: "balance", ability: 4}
    id: 10826
    pain_scale: 2
    progress_type: "onboarding"
    type_name: "psfs"
    updated_at: "2020-06-25T03:48:16.843Z"
    uuid: "02151b21-b45b-49c1-bd2a-647dddcd1f5a"

Y luego revienta con:

    Connection Error
    Something happened with the network

Cuando normalmente las peticiones siguientes debieron ser:

    /diseases/
    /pain_spots/
    /ongoing_drafts/
## Primera Sospecha

Como el `previous_progress_forms` retorna más de 3 *aggravating activities* el formulario actual parece explotar en alguna de las validaciones.

Lo creo así porque en frontend el error de conexión solo se levanta cuando en caso de que ningún error conocido sea capturado:

    /* navigation.action-creators.js, Line 35 */
    
    export function fetchPatientInfo({ patientId, uuid }) {
      /*(...)*/
      try {
        let previousForm = null;
        if (isProgressForm) {
          previousForm = (await api.get(`/forms/${uuid}/previous_progress_forms`)).data;
        }
      } catch (err) {
        if (err.response) {
            switch (err.response.status) {
              case 404:
                dispatch(resourceNotFound());
                break;
              case 422:
                dispatch(unprocessableEntity());
                break;
              default:
                dispatch(fetchPatientInfoFailure({ errorInfo: err.response.data }));
            }
          } else {
            dispatch(networkFailure()); // HERE
          }
          return null;
      }  
    }

**¿Qué puedo intentar para replicarlo?**
Tomar un form del mismo tipo en staging, y a su previous, crearle más actividades agravantes.

Formulario usado en staging https://forms.luna.farzoo.click/patients/9308da6a-1c14-48c4-a75e-df9e1a772468/forms/0ffabc2c-36e1-40f1-85a9-bda6c575ec35

**Y sí es esa la causa.**

![](https://paper-attachments.dropbox.com/s_DEDA15BAAB1FA29CE0955101B01C665460E05DB473FD34557577E245FE8BF9E0_1594335301106_Screen+Shot+2020-07-09+at+5.54.29+PM.png)


Una vez elimino la *aggravating activity* extra del *previous progress*, se resuelve el problema.

