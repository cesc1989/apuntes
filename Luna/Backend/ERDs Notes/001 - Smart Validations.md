# Smart Validations

Author: Francisco Mercedes

## Contexto

La aplicación móvil de Therapists permite crear Charts con pocas validaciones. Lo cual ha representado lo siguiente:

- Muchos errores a posteriori
- 44 mil chart fueron rechazadas en 2025
- Causando retrabajo para Therapists y el equipo de Luna

## Objetivo

Este proyecto tiene como objetivo implementar validaciones proactivas no intrusivas para capturar errores antes de crear los Charts.

Se enfoca en las causas más comunes de rechazos y busca mejoras de un 25% en entrega de Charts.

## Enfoque

Una query GQL que se dispara en tiempo real cuando se cambia el contenido del campo del Chart. Responde con retroalimentación para que el Therapist tome medidas al ir completando la Chart.

Este es un enfoque sencillo que tiene como contra que se podrían tornar lentas las validaciones según que tan complejas puedan llegar a ser a futuro.

### Alternativa: Validaciones en aplicaciones móviles

Positivos:
- Cero latencia
- Cero delay

Negativos:
- Implementación doble porque son dos aplicaciones nativas
- Solo se puede procesar lo que el dispositivo del therapist puede ejecutar
