# Plan General Inicial de Poffer Syncer

## Conclusiones Iniciales

Son tres proyectos en uno:

- Gema Pocket: Integrar API de Pocket (OAuth 2.0)
- Gema Hootsuite: Integrar API de Hootsuite (OAuth 2.0)
- RoR app: Integra las gemas y provee *callbacks* para OAuth 2.0

En términos de desarrollo:

- Gema para integración con Pocket
- Gema para integración con Hootsuite
- RoR app para unificar todo:
    - Horarios de publicación **en Base de Datos**
    - Sesión(Devise) para solo ingresar yo
    - Oauth2 para tener Callback URL para conectar con Hootsuite y Pocket
    - Trabajos en 2do plano (Sidekiq)
        - Cada 1hr (Sidekiq Scheduler)

**¿Vale la pena que sea la integración mediante gemas?**
Sí, para aprender de ese tipo de flujo en proyectos que son solo librerías:

- Build
- Rubygems
- Versionamiento
- Integrar en Rails
## Gema Pocket
- Gestión de API keys
- Pedir artículos con etiqueta *schedule*
- Extraer datos
- Archivar artículos
## Gema Hootsuite
- Gestión de API keys
- Control de *refresh token*
- Agendar publicación
## Stack
- RoR
- Tailwind o Bootstrap
- Devise
- Sidekiq/Sidekiq Scheduler
- Stimulus/View Component
- RSpec
- Heroku

