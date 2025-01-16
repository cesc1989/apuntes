# ✅ [SOLVED] Refactor: Caso Preguntas y Respuestas Demasiado Ligados
El refactor propuesto en el *pull request* https://github.com/lunacare/patient-forms-frontend/pull/312 intentó resolver un error donde un form que tiene X cantidad de preguntas y guarda X-1 respuestas, la cantidad de preguntas que se muestra en el borrador o la página de resultados es X-1 en vez de X.

Si un formulario de tipo KOOS que tiene 7 preguntas solo alcanza a guardar (por error de asíncrona) 6 respuestas, cuando se recarga el formulario solo listará 6 preguntas.

Eso ocurre porqué el código que dibuja las preguntas es muy dependiente con la respuesta del endpoint de las *answers.* Claramente no debería ser así.

Los elementos del endpoint de *questions* deben dibujarse independientemente de lo que haya en la respuesta del endpoint de *answers.* Lo propuesto en el PR está lejos de lo que en verdad se necesita y no soluciona el problema.

# Casos

Formulario con todas las preguntas y respuestas → https://forms.luna.farzoo.click/patients/463b9677-3dbf-47fb-b89d-c616c04d6af1/forms/4d0f4033-b7e8-b1aa-c9b4-c41ea00af1e9

Formulario con el error. Es del mismo tipo de arriba pero solo muestra 9 preguntas (porque tiene 9 respuestas) → https://forms.luna.farzoo.click/patients/flufflykins2/forms/03bd2442-8614-0ce7-3093-f9b1edc37501

**Corregido por** https://github.com/lunacare/patient-forms-frontend/pull/329 👏🏼 

