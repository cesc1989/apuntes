# RFC 001: Dominio Raíz o Subdominio para alojar la app

Antes de completar el lanzamiento de Enlacito me preguntaba qué debería elegir `app.enlacito.co` o `enlacito.co` para hospedar la aplicación.

Esta simple pregunta plantea algunos desafíos que no había considerado y que quiero dejar claros en este documento.

## A) Raíz: `enlacito.co`

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

#### Solución: Definir las rutas de cada `/PAGINA` de primero que la ruta que define `/CODE`

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

#### Solución: Expresión regular a `/:short_code` para que  ajuste preciso a códigos alfanumérico

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

#### Solución: Anidar rutas estáticas en scope o namespace para hacer su URL única

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


## B) Subdominio: `app.enlacito.co`

Si elijo montar la aplicación en el subdominio, va a pasar que los enlaces acortados serían:

```
https://app.enlacito.co/CODIGO
```

Y eso dista de ser lo que tengo en mente al recortar enlaces `https://enlacito.co/CODIGO`. A continuación detallo la situación con ayuda de DeepSeek.

### ¿Por qué en un subdominio?

Para poder tener esta configuración:

- `enlacito.co`: sitio web oficial, landing page, etc.
- `app.enlacito.co`: donde se usa la aplicación por los usuarios registrados.
- `enlacito.co/CODE`: URL recortada.

### ¿Qué ventaja tiene usar subdominios?

Si dejo la raíz libre, se puede usar cualquier otra tecnología de frontend para hacer la página web y sitios de marketing.

> [!Note]
> Un ejemplo de esto es Luna. Donde el dominio raíz apunta a la página de marketing hecha en PHP mientras que los subdominios apuntan a diferentes aplicaciones hechas en Rails, PHP, Elixir, JavaScript.

Esto libra a la aplicación Rails de tener toda la carga del servicio + página web. También será una buena forma de dividir la carga al dejar que la app Rails se encargue del core y hacer la página web en otra tecnología y montarlo en algún hospedaje diferente u otro VPS.

### ¿Cómo puedo apuntar la app al subdominio y que el enlace acortado use el dominio raíz?

Con lo anterior en mente, y siendo lo que sería ideal, cómo logro esta configuración donde:

- `app.enlacito.co`: es la aplicación para los usuarios registrados.
- `enlacito.co/CODE`: URL recortada.

En Zed, le pregunté a DeepSeek y sugirió hacer re-dirección con Nginx o usar un microservicio para las re-direcciones.

**Configuración de DNS**

Crear 3 registros A/AAAA o CNAME:

- `enlacito.co` -> IP/servidor de redirects
- `app.enlacito.co` -> IP/servidor de la app Rails
- `www.enlacito.co` -> Servidor estático (Netlify/Vercel/S3)

> [!Note]
> En mí caso el servidor de redirects y la app Rails serían el mismo.

**Servidor de Redirects (enlacito.co)**

Sugiere que cree un microservicio solo para los redirects. Esto implicaría otra app (Hanami, Sinatra) con acceso a la base de datos y que solo haga los redirects.

También sugirió usar Nginx para los redirects pero esto no es me llama la atención porque no podré hacer seguimiento para analítica.

Así sugiere configurar Nginx:
```bash
server {
    server_name enlacito.co;
    
    # Solo maneja paths que parezcan códigos (ajusta la regex)
    location ~* ^/[a-zA-Z0-9]{6,}$ {
        proxy_pass http://tu_app_rails;
        # O si quieres evitar Rails completamente:
        # rewrite ^/(.*)$ http://tu_app_rails/redirects/$1 permanent;
    }

    # Todo lo demás redirige al sitio estático
    location / {
        return 301 https://www.enlacito.co$request_uri;
    }
}
```

**Servidor Estático (www)**

Para esto se podría montar la página en Vercel, Netlify, GitHub Pages, S3, Cloudflare.

**Beneficios (según DeepSeek)**

- **Separación clara**: Cada servicio independiente
- **Escalabilidad**: Puedes escalar los redirects sin tocar el sitio estático
- **Seguridad**: Aislamiento de componentes
- **Performance**: Servir redirects es ultra rápido con Nginx directo

### Conclusión Opción B

Después de consultar con DeepSeek esta alternativa se ve mucho más trabajosa.

La clave de esta configuración:

- `enlacito.co` -> IP/servidor de redirects
- `app.enlacito.co` -> IP/servidor de la app Rails
- `www.enlacito.co` -> Servidor estático (Netlify/Vercel/S3)

es que puedo tener la raíz y el subdominio app en el mismo servidor y misma app Rails. ==La clave estará en hacer que si alguien navega a algo que no es una URL acortada, se redirige al sitio web estático `www.enlacito.co`.==

> [!tip]
> Recuerda que un dominio raíz puede apuntar a la IP del hospedaje o hacer un redirect a `www`. O viceversa.

Dicha redirección se puede controlar con Rails:
```ruby
def show
  # Si parece un código de link (ajusta el regex según tu formato)
  if params[:short_code] =~ /\A[a-z0-9]{6}\z/i
    link = Link.find_by(short_code: params[:short_code])
    
    link&.increment!(:clicks)
    
    redirect_to(link.long_url, allow_other_host: true) and return
  end
  
  # Si no es un código, redirige al sitio estático
  redirect_to "https://www.enlacito.co/#{params[:short_code]}"
end
```

Como con Nginx:
```bash
server {
    server_name enlacito.co;
    
    # Ruta del sitio estático (ajusta la ruta)
    root /var/www/enlacito-static;
    
    # Intenta servir archivos estáticos primero
    try_files $uri $uri.html $uri/ @rails;
    
    location @rails {
        # Solo pasa a Rails lo que parezca un código de enlace (ajusta el regex)
        if ($uri ~* "^/([a-z0-9]{6,})$") {
            proxy_pass http://tu_app_rails;
            break;
        }
        
        # Redirige todo lo demás al sitio estático (www)
        return 301 https://www.enlacito.co$request_uri;
    }
}

server {
    server_name www.enlacito.co app.enlacito.co;
    
    # Configuración para www (sitio estático)
    if ($host = www.enlacito.co) {
        root /var/www/enlacito-static;
        try_files $uri $uri.html $uri/ =404;
    }
    
    # Configuración para app (Rails)
    if ($host = app.enlacito.co) {
        proxy_pass http://tu_app_rails;
        # ... resto de configuración Rails
    }
}
```

# Decisión

A la fecha, 10 de Enero de 2026, el proyecto está corriendo bajo la opción A. Mantendré todo bajo el dominio raíz y usaré Rails para las páginas de mercadeo y aterrizaje.