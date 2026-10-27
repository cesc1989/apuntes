# Plan General de Lanzamiento

Tengo claro que no haré muchas características. Ahora mismo solo será:

- Acortar ✅
- Personalizar half-end ✅
- Personalizar Imagen open-graph ✅
- Analítica ✅

Una vez abra el registro al público, ¿cómo hago para que la gente llegue? Estas son algunas cosas que hacer.

> [!Important]
> Lo clave: ==🟢**la distribución importa más que el producto** al inicio.==
>
> **Enfócate primero en un nicho** (ej: pequeños negocios) y resuelve sus problemas específicos. La confiabilidad y velocidad son clave para que recomienden.

## Ideas

### Growth Marketing 🟡

- **Participa en foros** (Reddit, Stack Overflow, comunidades técnicas) donde se hable de productividad o herramientas, aportando valor y mencionando tu solución cuando sea relevante.
- **Distribución antes que features** 🌟
    - Publica en: Product Hunt, Indie Hackers, Reddit (subreddits de devs/marketing), X.
    - ==🟠Un post claro: problema → solución → link.==
- **ProductHunt + Betalist**: lanza el día 1 con captura de 30 s; regala 1 mes Pro a quien comente.

> [!Info]
> También en Shelv: https://www.shelv.io/
> 
> Aquí ya hice el post sobre enlacito.

- **Reddit + Foros**: aporta scripts o plantillas gratis en r/marketing, r/startups, BlackHatWorld; firma con tu link.

### Landing Pages para Posicionamiento 🚨

> [!Important]
> Esto es lo primero que puedo hacer y va de la mano de lo que estoy pensando.
> Las landings las puedo hacer con Claudio. Le puedo pedir que revise y compare otros sitios (los que encontré en Descubrimiento) y que haga unas páginas sencillas.

- **Casos de uso concretos**
    - "Para WhatsApp", "para campañas", "para redes sociales", "para emails".
    - Cada caso = una mini landing o sección.
- **SEO de nicho**: publica 5–10 artículos cortos que respondan “cómo acortar enlaces para…” (email, Twitter, WhatsApp, campañas SMS, afiliados). Indexa en Google y Bing el primer mes.

### Medir y Pedir Retroalimentación 🚨

> [!Info]
> Ya tenemos Umami configurado para la landing y también está el modelo `UserEvent` para seguir el paso de los usuarios en el servicio.

- **Mide desde el día 1**
    - Analytics + eventos (crear link, llegar al límite, click en upgrade) ✅

> [!Important]
> Creo que esto vale la pena hacerlo.

- **Itera con usuarios, no en silencio**
    - Un feedback link visible: _“¿Qué te falta?”_.


### Marketing de Influencers / Referidos 🟡

> [!Note]
> El primero es más sencillo de hacer. El 2do necesita más configuración, incluso desarrollo.

1. **Contacta a creadores de contenido pequeños/medianos** y ofréceles cuenta Pro gratis a cambio de feedback o mención.
2. **Programa de referidos**: 1 mes extra de Pro por cada usuario que traiga; automatiza con webhooks.


### Integraciones mediante API ❌

> [!Warning]
> No por ahora porque esto es desarrollo. A no ser que llegue algo que si al hacer esto signifique un negocio para el servicio.

- **Ofrece integración simple**: plugin para WordPress, extensión navegador.
- **Integraciones**: publica en Zapier, Pabbly y n8n
	- los flujos “acortar → spreadsheet → notificación” captan SaaS users.