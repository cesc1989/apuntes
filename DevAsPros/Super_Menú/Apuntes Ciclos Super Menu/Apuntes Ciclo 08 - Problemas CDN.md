# Apuntes Ciclo 08 - Problemas CDN

Después de varios cambios, como las subidas a Rails 7.2.3, Ruby 3.4.8, y actualizaciones de gemas cuando fui a probar cargar una imagen en producción me dio error.

## Configuración inválida de storage.yml con CloudFront y S3

Este error:
```
Aws::S3::Errors::AccessControlListNotSupportedThe
bucket does not allow ACLs (Aws::S3::Errors::AccessControlListNotSupported)
```

Y así estaba `config/storage.yml`:
```yaml
amazon:
  service: S3
  access_key_id: <%= ENV.fetch("SUPER_MENU_AWS_ACCESS_KEY_ID") %>
  secret_access_key: <%= ENV.fetch("SUPER_MENU_AWS_SECRET_ACCESS_KEY") %>
  bucket: <%= ENV.fetch("SUPER_MENU_AWS_S3_BUCKET_NAME") %>
  region: <%= ENV.fetch("SUPER_MENU_AWS_REGION") %>
  public: true
```

Explicó DeepSeek que la propiedad `public` no puede estar ahí porque el bucket no tiene permisos en el ACL.

Una vez lo quité y probé a cargar imágenes todo fue correcto.

## Helper de carga de imagen en CDN desactualizado

Parece ser que este otro error dio al pasar a versiones más recientes de Rails. Como no probé SuperMenu a fondo no me di cuenta.

El error daba `ArgumentError` si cargaba desde el endpoint o `ActionView::Template::Error` si era desde la vista donde se intentaba cargar una imagen que está en el CDN.

El error en detalle fue este:
```
missing keywords: :expires_in, :filename, :disposition, :content_type (ArgumentError)
```

Por esta línea de código en el router:
```ruby
blob.service.url(blob.key).sub(
```

Esta es la firma de Service#url:
```ruby
url(key, **options)
```

La solución estuvo por pasar los keywords faltantes a la función:
```ruby
blob.service.url(
	blob.key,
	expires_in: 1.hour,
	filename: blob.filename,
	disposition: :inline,
	content_type: blob.content_type
).sub(
```

Con eso puesto ya las imágenes cargaban normalmente.