# Registrar Eventos de Usuarios para Entender el Uso

Esto lo sugirió Gepeto para saber si el producto lo usan o en qué momento podrían pasar a pagar o no. Copio y pego lo que me dijo.

### Regla de oro

> **Cada métrica debe responder una pregunta clara.**  
> Si no sabes qué decisión tomarías según el número, no la midas.

### Qué medir (lo mínimo imprescindible)

No intentes medir todo. Solo lo que te ayuda a decidir.

1. **Activación**
    - Usuarios que crean **al menos 1 link**.
    - Usuarios que crean **2 links** (límite del plan free).  
        → Esto te dice si el producto “hace clic”.
2. **Conversión**
    - Cuántos usuarios llegan al **límite del plan gratuito**.
    - Cuántos hacen clic en **“Upgrade a Pro”**.
    - Cuántos **pagan** realmente.
3. **Uso real**
    - Cuántos links se crean por usuario.
    - Cuántos clicks reciben esos links.  
        → Si nadie vuelve, no hay valor percibido.
4. **Drop-off**
    - Landing → registro
    - Registro → primer link
    - Primer link → segundo link  
        → Aquí es donde normalmente se pierde la gente.

### Eventos concretos que deberías tener

Con 6–8 eventos basta:

- `signup_completed`
- `link_created`
- `free_limit_reached`
- `upgrade_clicked`
- `payment_started`
- `payment_completed`
- (opcional) `link_clicked`

### Herramientas simples (no te compliques)

- **Plausible / Umami** → métricas de tráfico y eventos simples.
- **PostHog** → funnels y sesiones (muy útil al inicio).
- **Stripe Dashboard** → ingresos y churn (no reinventes esto).

> [!Important]
> Aquí la ganadora es Umami que tiene plan gratuito y además la puedo auto hospedar con Pikapods.

### Cómo usar esos datos (lo importante)

- Si muchos **llegan al límite pero no pagan** → problema de precio o propuesta.
- Si pocos **crean el segundo link** → el producto no engancha.
- Si hay tráfico pero no registros → la landing no convence.
- Si pagan y no vuelven → no hay hábito ni necesidad real.