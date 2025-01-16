# Problema ASES Pain Scale en API V2
Progress UUID -> 4c55cf41-70b5-4d28-abfc-cf0ebb3135b0
Progress care plan -> 3e38e1a8-5018-44c6-a46b-e107e108b823
Onboarding Summary -> https://forms.lunacare.co/patients/a425bedd-dcad-45d1-a596-9a2d3e16dcb3/summary/5dc09900-8f1b-43b9-837e-e6c4de6415d1

ASES con todas las respuestas hasta la pain scale... aunque la pain scale salió en una parte que era incorrecta? en staging me dio la página 4 para llegar ahí.

El tema es que en stagin la paginacion por defecto es de 10 y al final de la página 1 empieza
el encabezado de la pregunta Pain Scale. Situacion similar a paginar preguntas sin secciones pero que tienen encabezado.


    "cursor": {
        "category_id": 3,
        "page": 1,
        "per": 10,
        "question_id": 30
    },
    
    pp V2::FormCursor.new(Form.find_by(uuid: '486d5d41-34e7-4484-b2c5-e29eda6fe14e')).find

SÍ FUE POR QUE LA PAGINACION EN STAGING ES DE 10 EN 10. En local funcionó bien de 3 en 3 de páginacion.

Pero sí hay un error.

La última pregunta la indica como si estuviera en la página 4 y no es así, está en la última. Pasa por la cabecera que está antes de la pregunta Pain Scale.

