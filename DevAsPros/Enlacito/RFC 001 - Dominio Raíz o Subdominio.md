# RFC 001: Dominio Raíz o Subdominio para alojar la app

Antes de completar el lanzamiento de Enlacito me preguntaba qué debería elegir `app.enlacito.co` o `enlacito.co` para hospedar la aplicación.

Esta simple pregunta plantea algunos desafíos que no había considerado y que quiero dejar claros en este documento.

## Raíz: `enlacito.co`

Si elijo montar la aplicación en la raíz del dominio, tendré que hacer la landing y páginas de marketing en Rails. Eso creo.

La cosa sería algo como:

- `enlacito.co` sin iniciar sesión: página inicial, presentación, landing page.
- `enlacito.co` sesión iniciada: aplicación general, administración de recursos y cuenta.
- `enlacito.co/CODE`: URL recortada.


## Subdominio: `app.enlacito.co`

Si elijo montar la aplicación en el subdominio, va a pasar que los enlaces acortados serían:

```
https://app.enlacito.co/CODIGO
```

Y eso dista de ser lo que tengo en mente al recortar enlaces `https://enlacito.co/CODIGO`.

### ¿Por qué en un subdominio?

Para poder tener:

- `enlacito.co`: sitio web oficial, landing page, etc.
- `app.enlacito.co`: donde se usa la aplicación por los usuarios registrados.
- `enlacito.co/CODE`: URL recortada.

### ¿Qué ventaja tiene usar subdominios?

Si dejo la raíz libre, se puede usar cualquier otra tecnología de frontend para hacer la página web y sitios de marketing.

> [!Note]
> Un ejemplo de esto es Luna. Donde el dominio raíz apunta a la página de marketing hecha en PHP mientras que los subdominios apuntan a diferentes aplicaciones hechas en Rails, PHP, Elixir, JavaScript.

### ¿Cómo puedo apuntar la app al subdominio y que el enlace acortado use el dominio raíz?

Con lo anterior en mente, y siendo lo que sería ideal, cómo logro esta configuración donde:

- `app.enlacito.co`: donde se usa la aplicación por los usuarios registrados.
- `enlacito.co/CODE`: URL recortada.

En Zed, le pregunté a DeepSeek y sugirió hacer re-dirección con Nginx o usar un microservicio para las re-direcciones.

