# Technical Plans: ¿cómo y por qué?
En este documento pondré varias fuentes que explican cómo escribir planes técnicos, cómo escribirlos, qué tener en cuenta, algunos ejemplos y guías.

## Recursos
- [Simple tricks to level up your technical design docs in 2024](https://careercutler.substack.com/p/simple-tricks-to-level-up-your-technical)
- [Conversación con ChatGPT 3.5](https://chat.openai.com/share/9d369ac5-350e-4da5-bc70-b2cc7c0b80d9)


# Why design docs / technical plans are useful and how they help you
## Reducir el riesgo
> By identifying potential challenges and risks upfront, a technical plan allows for proactive mitigation strategies to be put in place.


## Alineamiento del equipo
> It ensures that everyone involved in the project is on the same page regarding the approach, methodologies, and technologies to be used.


## Comunicación
> It serves as a communication tool, (…) within the project team (…). A (…) technical plan clearly articulates the project scope, requirements, timelines, and deliverables.



# Recomendaciones
## Mantenlo al grano

Escribe solo lo necesario para tomar una decisión.

> Only include enough information to make the right decision on the approach.
> 
> It’s about solving the ambiguity and aligning on the path forward.

Incluye detalles de partes complejas o de mucho riesgo:

- *API route data changes*
- *Data models*
- *Database schema and indexes*


## Consigue primero retroalimentación de personas clave
> You want to get feedback from the most vocal reviewers individually first.
> 
> Doing this gets those vocal reviewers bought in to your doc because you incorporate their feedback early on.

En mi caso en Luna, si Ryan/Jason revisa y está de acuerdo, voy ganando.


## Entiende por qué cada sección es necesaria

Sigue siempre una plantilla pero entiende porqué cada sección es necesario y añade solo lo que ayude a mover la decisión. Si no agrega valor para tomar la decisión, quitalo.

**About this doc**

1. **What it has:** Status of the doc and signoffs needed to move forward.
2. **Why:** Make it clear right away whether this doc still needs work to move on.

Esto lo hace Jason en los documentos que tenemos

![](https://paper-attachments.dropboxusercontent.com/s_3F2726D0798B44D211A404EA87E9ACDC6312442303DBC4280E5539D7B33889D6_1706479406230_imagen.png)


**Context**

1. **What it has:** Problem being solved and the impact.
2. **Why:** Understand why this is being worked on.

**High level approach**

1. **What it has:** Overview of how you plan to approach the problem. 
    *Note****:*** *include a visual of how the feature should work from a user perspective and/or a diagram of your data flow here.*
2. **Why:** Allow people to skim the doc and quickly understand your approach.

**Technical Design Requirements**

1. **What it has:** High level requirements from a technical perspective to support the user use cases.
2. **Why:** Make it clear before jumping into the detailed design **why** the detailed design is needed from a technical perspective.
    *Note: rarely seen in design docs but I believe is valuable.*

**Detailed design**

1. **What it has:** The API, interface, model updates and anything complex.
2. **Why:** Agree on the essential pieces of the code updates.

**Risks**

1. **What it has:** Ways the project may not achieve its goals and how to mitigate.
2. **Why:** Reduce risk by thinking of it ahead of time and calling out mitigations. Show your reviewers you are thinking of this so they gain more trust in you.

**Test plan**

1. **What it has:** How the solution will be tested to ensure it works as expected.
2. **Why:** Reduce risk by thinking ahead on how it will be tested. Improve accuracy of your estimate.

**Rollout plan**

1. **What it has:** Timeline breakdown of rollout to customers.
2. **Why:** Reduce risk by gradually rolling out to customers and aligning everyone on the plan.


# Planes que ya he redactado

Estos son planes que ya he escrito para diferentes proyectos en Luna. No aplican todo lo que se menciona anteriormente.

## Credentialing to Symplr
https://www.dropbox.com/scl/fi/zj6tgpynl5bmb0db67cbz/Technical_Plan_-_Therapist_to_Symplr_Integration.pdf?dl=0&rlkey=273umq9enb6atiurn87si68f6



## Symplr to Hubspot
https://www.dropbox.com/scl/fi/fxie74bgztg69uxe294ft/Technical_Plan_-_Symplr_to_Hubspot_Integration.pdf?dl=0&rlkey=8quixlbblrw5cjvuoved3a6az



## Luxe to Symplr
https://www.dropbox.com/scl/fi/wl7iu7of428h75t0r4mi8/Technical_Plan_-_Luxe_to_Symplr_Integration.pdf?dl=0&rlkey=hhfd6heplqk5qii4cje8wjavi



## Bye Athena
https://www.dropbox.com/scl/fi/9836b7te0h3w1jm6p03bo/23-08-29-Bye-AWS-Athena-Technical-Plan.pdf?dl=0&rlkey=64flnbwpe6a0yiumtcvn342ka


