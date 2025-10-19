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

### Conclusión

La implementación fue sencilla y la cabecera responde lo que es pero veo que no cachea todo. En todo caso hay que devolver las imágenes desde un CDN para que sea más rápida esa parte.

## Configura CDN para cargar las imágenes del menú

> [!Note]
> Para esta iteración de Super Menú configuré el CDN en modo directo.

Para lograr esto hay dos opciones:

- Configurar el origen del CDN un bucket de S3
	- Forma "direct URL exposure"
- Configurar el origen del CDN el servidor de la aplicación
	- Forma Proxy

Para la primera solo hay que configurar el helper en `routes.rb` para que las imágenes carguen directo desde el CDN.

Para la segunda opción es que la recomendación de las guías sobre el setting es que se debe usar.

### Configuración de CloudFront y S3

Seguí el [tutorial](https://headey.net/rails-assets-active-storage-and-a-cloudfront-cdn) y en la parte del policy en el bucket ya AWS se encarga de actualizar eso por mí. No tuve que modificar la policy.

En todo caso ese tutorial me sirvió solo en gran parte. No funcionó totalmente para tener el CDN configurado.

La forma más fácil para configurar la distribución en CloudFront es usar el origen que sea el bucket en S3. Cuando se hace de esa forma hay que prestar atención al detalle del OAC (Origin Access Control). La clave aquí es elegir lo recomendado por CF.

Para la policy en el bucket CloudFront se encargará de hacer la actualización directamente pero hay una salvedad que varios tutoriales mencionaron.

Esta es la policy que agrega CloudFront:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowCloudFrontServicePrincipal",
            "Effect": "Allow",
            "Principal": {
                "Service": "cloudfront.amazonaws.com"
            },
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::supermenu-pruebas/*",
            "Condition": {
                "ArnLike": {
                    "AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT_ID:distribution/E12GR9OR9JGS21"
                }
            }
        }
    ]
}
```

Sin embargo, los tutoriales recomiendan cambiar la condición `ArnLike` por `StringEquals`. De esa forma fue que deje al final la policy.

### Configuración de Active Storage

En el [tutorial de Avo](https://avohq.io/blog/cdn-for-active-storage-uploads) es donde explican mejor esta configuración de CloudFront como CDN para archivos cargados en S3 mediante Active Storage.

Explican que Active Storage tiene dos modos de trabajar:

- Redirect
- Proxy

#### Redirect

El modo Redirect (por defecto) hace que al pedir una imagen AS retorne una URL que redirecciona a la URL real del recurso. Es una forma de indirección para proteger la URL real del recurso.

El modo Redirect tiene la desventaja de que no se puede cachear la respuesta porque hay una redirección. Por lo tanto no se integra bien con CDN.

Ilustración de Avo:

![[as.redirect.mode.webp]]

Para completar la configuración en este modo solo basta con concatenar el ID del blob con la URL de la distribución. Así lo muestra Avo:
```ruby
direct :cdn_image do |model|
  if model.respond_to?(:signed_id) # We're dealing with an ActiveStorage::Blob
    File.join(ENV["CDN_URL"], model.key)
  else
    File.join(ENV["CDN_URL"], model.blob.key)
  end
end
```

Y esta es la forma que lo hizo Claudio:
```ruby
# Generate direct S3 URL and replace S3 domain with CloudFront domain
direct :cdn_image do |model, options|
	if model.respond_to?(:blob)
		# For attachments get the blob and generate S3 URL
		blob = model.blob
		blob.service.url(blob.key).sub(
			/https:\/\/[^\/]+/,
			"https://d39m631mxio8iw.cloudfront.net"
		)
	else
		# For variants
		model.service.url(model.key).sub(
			/https:\/\/[^\/]+/,
			"https://d39m631mxio8iw.cloudfront.net"
		)
	end
end
```

Son lo mismo pero el de Avo es más sencillo y al grano. Avo menciona dos desventajas de esta forma:

**La URL es pública y permanente.** Ejemplo: `https://d39m631mxio8iw.cloudfront.net/vq2zcb111ar8j9ely2ioz3192vo9`. Esa URL no va a cambiar aunque se elimine el objeto del bucket.

**No se puede aplicar control de Acceso**

#### Proxy

> [!Tip]
> Esta es la forma en que las guías de Rails sugieren hacer esta configuración.

El modo Proxy funciona como un intermediario entre el CDN y el bucket en S3. En este modo la primera petición el CDN no tendrá el archivo así que el servidor debe pedirlo a S3 y devolverlo al CDN para que sea cacheado y responder al cliente.

En la siguiente petición el CDN será quien entregue el archivo al ya estar cacheado.

Ilustración de Avo:

![[active-storage-proxy-mode.webp]]

Las guías de Active Storage mencionan en la sección Proxy Mode:

> files can be proxied instead. means your application servers will download file data from the storage service in response to requests. useful for serving files from a CDN.

Y agregan esta configuración:
```ruby
# config/initializers/active_storage.rb
Rails.application.config.active_storage.resolve_model_to_route = :rails_storage_proxy
```

Así lo había configurado en `production.rb`:
```ruby
config.active_storage.resolve_model_to_route = :rails_storage_proxy
```

