# Plan General de Lanzamiento

> [!Note]
> Estrategias complementarias:
> - [[02-Plan Redes Sociales Enlacito]]
> - [[Reddit Marketing]]

Tengo claro que no haré muchas características. Ahora mismo solo será:

- Acortar ✅
- Personalizar half-end ✅
- Personalizar Imagen open-graph ✅
- Analítica ✅

Una vez abra el registro al público, ¿cómo hago para que la gente llegue? Estas son algunas cosas que hacer.

> [!Important]
> Lo clave: ==🟣**la distribución importa más que el producto** al inicio.==
>
> **Enfócate primero en un nicho** (ej: pequeños negocios) y resuelve sus problemas específicos. La confiabilidad y velocidad son clave para que recomienden.

## Ideas

### Landing Pages para Posicionamiento 🚨

> [!Important]
> Esto es lo primero que puedo hacer y va de la mano de lo que estoy pensando.
> Las landings las puedo hacer con Claudio. Le puedo pedir que revise y compare otros sitios (los que encontré en Descubrimiento) y que haga unas páginas sencillas.

- **Casos de uso concretos**
    - "Para WhatsApp", "para campañas", "para redes sociales", "para emails".
    - Cada caso = una mini landing o sección.
- **SEO de nicho**: publica 5–10 artículos cortos que respondan “cómo acortar enlaces para…” (email, Twitter, WhatsApp, campañas SMS, afiliados). Indexa en Google y Bing el primer mes.

### Medir y Pedir Retroalimentación 🚨

> [!Note]
> Ya tenemos Umami configurado para la landing y también está el modelo `UserEvent` para seguir el paso de los usuarios en el servicio.

- **Mide desde el día 1**
    - Analytics + eventos (crear link, llegar al límite, click en upgrade) ✅

> [!Important]
> Creo que esto vale la pena hacerlo. Un botón o algo para que cualquiera pueda dar feedback.

- **Itera con usuarios, no en silencio**
    - Un feedback link visible: _“¿Qué te falta?”_.


### Marketing de Influencers / Referidos 🟡

> [!Note]
> El primero es más sencillo de hacer. El 2do necesita más configuración, incluso desarrollo.

1. **Contacta a creadores de contenido pequeños/medianos** y ofréceles cuenta Pro gratis a cambio de feedback o mención.
2. **Programa de referidos**: 1 mes extra de Pro por cada usuario que traiga; automatiza con webhooks.


### Growth Marketing 🟡

- **Participa en foros** (Reddit, Stack Overflow, comunidades técnicas) donde se hable de productividad o herramientas, aportando valor y mencionando tu solución cuando sea relevante.
- **Distribución antes que features** 🌟
    - Publica en: Product Hunt, Indie Hackers, Reddit (subreddits de devs/marketing), X.
    - ==🟠Un post claro: problema → solución → link.==
- **ProductHunt + Betalist**: lanza el día 1 con captura de 30 s; regala 1 mes Pro a quien comente.

> [!Note]
> También en Shelv: https://www.shelv.io/
> 
> Aquí ya hice el post sobre enlacito.

- **Reddit + Foros**: aporta scripts o plantillas gratis en r/marketing, r/startups, BlackHatWorld; firma con tu link.


### Integraciones mediante API ❌

> [!Warning]
> No por ahora porque esto es desarrollo. A no ser que llegue algo que si al hacer esto signifique un negocio para el servicio.

- **Ofrece integración simple**: plugin para WordPress, extensión navegador.
- **Integraciones**: publica en Zapier, Pabbly y n8n
	- los flujos “acortar → spreadsheet → notificación” captan SaaS users.


## Plan de 2 o 3 semanas 🔑

> [!Note]
> Esto es lo que voy a hacer en dos o tres semanas mientras la situación en OrderlyMeds se va definiendo, al mismo tiempo que le voy sacando provecho a Claudio.

> [!Important]
> Continuar haciendo lo que vengo haciendo según lo definido en [[02-Plan Redes Sociales Enlacito]].
>
> Ajusta la ejecución para seguir más las ideas expuestas en el Plan de Redes Sociales.


### Continuar lo de Plan de Redes Sociales 🟡

> [!Important]
> La clave aquí es dedicarle tiempo a los posts para que sean más a consciencia y con ganas.

Como es lo más fácil de hacer porque los contenidos los puedo sacar de mis lecturas en Feeder debo continuar con lo detallado en [[02-Plan Redes Sociales Enlacito#Plan de Contenidos y Ejemplos]]. Con un giro. Así:

- **Bluesky**: posts de todo lo que lea.
- **Mastodon**: posts de temas anti-IA y programación. También de cultura general.
- **Twitter**: Tocaría postear manualmente pero puedo tomar lo que publique en Bsky para ello.
- **LinkedIn**: herramientas, temas de curiosidades (Xataka, Microsiervos)

> [!Warning]
> En Twitter recuerda el 1er post debe ser una imagen. La primera respuesta del hilo que sea el enlace.


### Nuevas Páginas Individuales y Nuevo Footer

Hay que cambiar el footer para que se parezca al de DAP, TinyURL o Tinny. La idea es que pueda haber columnas de enlaces a diferentes subpaginas. Entonces podría haber una columna "Características" con enlaces a subpáginas de cada característica.

Esto sería una oportunidad para tener más enlaces, más paginas rankeables y más contenido indexable.

> [!Note]
> Hacerlo como hace TinyURL o Tinny.
>
> ![[footer.tinyurl.png]]
>
> ![[footer.tinny.png]]


### Landing Pages de Casos de Uso

Que sean como ejemplos de cómo usar y para qué se puede usar Enlacito. Creo que de primerazo se pueden hacer landings para:

- Enlaces en descripción de vídeos de YT
- Hacer publicaciones en Redes Sociales
- Campañas por WhatsApp