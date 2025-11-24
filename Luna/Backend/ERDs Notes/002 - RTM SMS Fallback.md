# RTM SMS Fallback

Autor: Victor Garzel
Proyecto: Grimoire

## Problema

Los pacientes no responden a las preguntas in-app de RTM.

## Propuesta

Enviar un SMS a estos pacientes con la pregunta que no respondieron.

## Objetivo

Incrementar la tasa de respuestas de RTM.

## Implementación

- Usar SMS cuando sea conveniente
- Enviar el SMS 24 horas después si no responden la pregunta in-app
- Si el paciente no responde la pregunta 1, al día siguiente se envía mediante SMS. Ese mismo día se muestra la pregunta 2 in-app.

### Enfoque

- Usando Twilio:
	- Ya tiene las preguntas
	- Manejas las respuestas y se conecta con Grimoire

### Rate Limits

- Twilio tiene como límite 20 peticiones por segundo por cuenta.
- Se controla a 18 peticiones por segundo mediante Oban.