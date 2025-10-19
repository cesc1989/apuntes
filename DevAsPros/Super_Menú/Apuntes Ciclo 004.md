# Apuntes Ciclo 004

## Cacheando la respuesta del controlador

Ver [[001 - Mejorar Carga de Fotos de Platos#Cachear respuesta JSON del controlador]]

Voy a probar si esto sirve. Cuando pruebo una petición normal veo estas cabeceras:

```
Cache-Control max-age=0, private, must-revalidate
```

Con el cambio aplicado según el RFC responde así el servidor Rails:
```
Cache-Control max-age=300, public
```

O sea que sí está haciendo lo que dijo gepeto.

### ¿Qué hace `Cache-Control`?

Ver [MDN](https://developer.mozilla.org/es/docs/Web/HTTP/Reference/Headers/Cache-Control).

> contiene _directivas_ (instrucciones) — tanto en peticiones como en respuestas — para controlar el almacenamiento temporal caching en navegadores y cachés compartidas (p. ej. Proxies, CDNs).

`max-age`
> Indica que las cachés pueden almacenar esta respuesta y reutilizarla para las peticiones subsecuentes mientras estas son recientes.

`public`
> La directiva `public` indica que la respuesta puede ser almacenada en un cache compartido.

### Alternativa: action caching

Podría probar con la gema [actionpack-action_caching](https://github.com/rails/actionpack-action_caching) si esto no brinda beneficios.

Las guías mencionan brevemente [action-caching](https://guides.rubyonrails.org/v7.1/caching_with_rails.html#action-caching).

