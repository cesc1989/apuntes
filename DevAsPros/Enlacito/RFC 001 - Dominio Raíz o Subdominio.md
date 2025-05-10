# RFC 001: Dominio Raíz o Subdominio para alojar la app

Antes de completar el lanzamiento de Enlacito me preguntaba qué debería elegir `app.enlacito.co` o `enlacito.co` para hospedar la aplicación.

Esta simple pregunta plantea algunos desafíos que no había considerado y que quiero dejar claros en este documento.

## Raíz: `enlacito.co`

Si elijo montar la aplicación en la raíz del dominio, tendré que hacer la landing y páginas de marketing en Rails. Eso creo.

La cosa sería algo como:

- `enlacito.co` sin iniciar sesión: página inicial, presentación, landing page.
- `enlacito.co` sesión iniciada: aplicación general, administración de recursos y cuenta.
- `enlacito.co/CODE`: URL recortada.

En este escenario las rutas quedarían así:

- `/users/sign_in`: inicio de sesión
- `/users/sign_up`: registro
- `/`: landing page, página principal
	- `/PAGINA`: otras páginas de información, contacto, etc.
- `/links`: listado de enlaces
- `/links/new`: formulario de nuevo enlace
- `/CODE`: redirección de enlace recortado

### Problema de usar solo el dominio raíz

La principal fuente de conflicto creo que serían las rutas `/CODE` vs `/PAGINA`. A continuación, algunas formas de reducir los posibles conflictos de esta implementación.

**🤔 Definir las rutas de cada `/PAGINA` de primero que la ruta que define `/CODE`**

De esta forma:
```ruby
Rails.application.routes.draw do
	# Rutas de páginas
	get "/contact", to: "about#contact"
	get "/team", to: "about#team"

  # (...)
  
  # Ruta que define redireccionamiento
  get "/:short_code", to: "redirects#show", as: :redirect
end
```

De esa forma siempre se encontrará primero la ruta más específica (la de una página). Cuando se navegue a una URL recortada, no se hallará en ninguna ruta de página y llegará al final para que sea procesada por el controlador `Redirects`.

**🤔 Expresión regular a `/:short_code` para que  ajuste preciso a códigos alfanumérico**

Así:
```ruby
Rails.application.routes.draw do
  # Páginas estáticas

  # (...)

  get "/:short_code",
    to: "redirects#show",
    as: :redirect,
    constraints: { short_code: /[a-zA-Z0-9]{6,}/ }
end
```

Esta sería una forma de tratar de que la ruta sea más específica pero igual habría que definir las páginas estáticas de primero porque esa expresión regular podría capturar rutas como `/contacto` o `/precios`.

Sirve pero no es la solución definitiva.

**🤔 Anidar rutas estáticas en scope o namespace para hacer su URL única**

Lo que podría ser la solución definitiva es anidar las rutas de las páginas estáticas en un scope/namespace y así todas las URLs serían únicas y no entrarían en la captura de la ruta de los short_codes.

```ruby
Rails.application.routes.draw do
  scope "/pages" do
	  get "/caracteristicas", to: "pages#caracteristicas", as: :caracteristicas
	  get "/contacto", to: "pages#contacto", as: :contacto
	  # otras rutas estáticas...
	end

  # (...)

  get "/:short_code",
    to: "redirects#show",
    as: :redirect,
    constraints: { short_code: /[a-zA-Z0-9]{6,}/ }
end
```

> [!Tip]
> En todo caso podemos mantener la expresión regular para más precisión de la ruta de redirección.

Usando `scope` les damos un agrupador para diferenciarlas que solo afectará la URL. No habrá que crear un módulo `Pages` y los helpers serían sencillos.

Tanto `scope` como `namespace` pueden servir para esto. Siendo `scope` lo más sencillo.


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

