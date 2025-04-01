# Lecciones Ruby - Therapists

# Reemplazar caracteres en un string por otra cosa

Supongamos que quiero que en la siguiente cadena

    "Alberto De la Espriella"

los espacios sean cambiados por guiones medios o bajos:

    "Alberto_De_la_Espriella"
    
    "Alberto-De-la-Espriella"

Se puede hacer de dos formas. Una es con `[String#gsub](https://ruby-doc.org/core-2.5.3/String.html#method-i-gsub)` y la otra `[String#tr](https://ruby-doc.org/core-2.5.3/String.html#method-i-tr)`:
```ruby
"Alberto De la Espriella".gsub(' ', '_') # => "Alberto_De_la_Espriella"
"Alberto De la Espriella".tr(' ', '_') # => "Alberto_De_la_Espriella"
```


# Creando un Digest MD5 para archivo en AWS

Korey hizo un cambio en la clase `app/services/s3_attachments_folder_service.rb` donde quería crear un digest MD5 para la carpeta que contiene los archivos del terapeuta:
```ruby
def create
	digest = Digest::MD5.file(@folder_name).base64digest
	@s3_client.put_object(bucket: @bucket, key: "#{@folder_name}/", content_md5: digest)

	update_therapist
end
```

> Documentación sobre [Digest::MD5.file](https://ruby-doc.org/stdlib-2.7.1/libdoc/digest/rdoc/Digest/Instance.html#method-i-file)

Sin embargo, esto da error porque `@folder_name` no existe como archivo. Es solo un string.

    Errno::ENOENT (No such file or directory @ rb_sysopen - siete_serre_2023-01-04):

**¿Qué es un [Digest MD5](https://en.wikipedia.org/wiki/MD5)?**

Es una función hash que devuelve un hash de 128 bits. Se puede usar como una suma de chequeo para asegurarnos que un archivo no ha sido corrompido.

Alternativa para solucionar el error:
```ruby
digest = Digest::MD5.hexdigest(@folder_name)
```

pero da este error:
```ruby
Aws::S3::Errors::InvalidDigest (The Content-MD5 you specified was invalid.):
```

Probé con:
```ruby
digest = Digest::MD5.base64digest(@folder_name)
```

pero da este otro error:
```ruby
Aws::S3::Errors::BadDigest (The Content-MD5 you specified did not match what we received.)
```

La documentación de [put_object](https://docs.aws.amazon.com/sdk-for-ruby/v2/api/Aws/S3/Client.html#put_object-instance_method) sobre `content_md5`.

    :content_md5 (String) — The base64-encoded 128-bit MD5 digest of the message (without the headers) according to RFC 1864. This header can be used as a message integrity check to verify that the data is the same data that was originally sent.
    
    Although it is optional, we recommend using the Content-MD5 mechanism as an end-to-end integrity check.

