# Mover archivos de una carpeta a otra en el mismo bucket

Para el cambio de nombre de carpeta en Therapist Credentialing tocaba mover los archivos a la nueva carpeta de los ya existentes.

Esto fue lo que intenté y también cómo logré hacerlo.


# Lo que intenté que no funcionó

Enlaces:

- [copy_to](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/S3/Object.html#copy_to-instance_method) de Aws::S3::Object
- [move_to](https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/S3/Object.html#move_to-instance_method) de Aws::S3::Object 

Cuando intenté con esos no logré pasar del error:

    The specified key does not exist. (Aws::S3::Errors::NoSuchKey)

Lo hice de esta forma:
```ruby
s3_client = Aws::S3::Client.new(
  access_key_id: ENV['AWS_ACCESS_KEY_ID'],
  secret_access_key: ENV['AWS_SECRET_ACCESS_KEY'],
  region: 'us-east-1'
)

bucket = ENV.fetch('AWS_S3_BUCKET_NAME')

s3_object = Aws::S3::Object.new(bucket, "#{source_folder}#{key}", client: client)

s3_object.move_to(bucket: bucket, key: "holamundocruelhola/")

s3_object.copy_to(bucket: bucket, key: 'holamundocruelhola/')
```


# Lo que terminó funcionado

Le tuve que preguntar a Chat GPT y me dio esto:
```ruby
source_folder = "ioaso_mando_2023-03-13"
destination_folder = "nojodanovalespaen"
s3_client = Aws::S3::Client.new(access_key_id: ENV['AWS_ACCESS_KEY_ID'], secret_access_key: ENV['AWS_SECRET_ACCESS_KEY'], region: 'us-east-1')
bucket = ENV.fetch('AWS_S3_BUCKET_NAME')

s3_client.list_objects_v2(bucket: bucket, prefix: source_folder).contents.each do |object|
  source_key = object.key
  destination_key = source_key.gsub(source_folder, destination_folder)
  s3_client.copy_object(
	bucket: bucket,
	copy_source: "#{bucket}/#{source_key}",
	key: destination_key
  )
end
```

Las primeras 4 líneas son la configuración y chat gpt me dio la respuesta en el uso de `list_objects_v2`.

