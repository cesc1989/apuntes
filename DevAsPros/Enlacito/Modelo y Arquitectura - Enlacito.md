# Modelo de Datos para Enlacito

Este sería el posible modelo para la tabla que guardará los enlaces:

```sql
CREATE TABLE urls (
  id SERIAL PRIMARY KEY,
  short_code VARCHAR(10) UNIQUE NOT NULL,
  long_url TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  clicks INTEGER DEFAULT 0
);
```

Otros modelos -> [[Otros Acortadores#Open Source]]

## Open Source

### Dub

Repo -> https://github.com/dubinc/dub

Hecho en NextJS y TypeScript.

El esquema de Dub es más completo. Aquí se pueden ver todos los esquemas -> https://github.com/dubinc/dub/tree/main/packages/prisma/schema

### YLL

Repo -> https://github.com/davidesantangelo/yll

Hecho en Ruby on Rails. Este es el esquema. Solo tiene una tabla.

```ruby
  create_table "links", force: :cascade do |t|
    t.string "url"
    t.string "password_digest"
    t.datetime "expires_at"
    t.string "code"
    t.integer "clicks", default: 0
    t.datetime "created_at", null: false
    t.datetime "updated_at", null: false
    t.index ["clicks"], name: "index_links_on_clicks"
    t.index ["code"], name: "index_links_on_code", unique: true
    t.index ["url"], name: "index_links_on_url"
  end
```

El campo `password_digest` es para generar enlaces protegidos por contraseña.

### RailsUrlShortener

Repo -> https://github.com/a-chacon/rails-url-shortener/

Hecho en Ruby on Rails. Funciona como engine. Este proyecto define varias tablas:

```ruby
create_table :rails_url_shortener_urls do |t|
	# optional if you can link it to a user or other model
	t.references :owner, polymorphic: true, null: true
	t.text :url, null: false, length: 2048 # the real url
	t.string :key, limit: 10, null: false # the unique key
	t.string :category # category for short url
	t.datetime :expires_at # valid until date for expirable urls
	t.timestamps
end

create_table :rails_url_shortener_visits do |t|
	t.belongs_to    :url
	t.string        :ip # client ip
	t.string        :browser # browser from user_agent
	t.string        :browser_version # browser version from user_agent
	t.string        :platform # platform from user_agent
	t.string        :platform_version # platform version from user_agent
	t.boolean       :bot # if the request was a bot or not
	t.string        :user_agent
	
	# variable where we save all data that can be catch from the request
	t.text          :meta
	t.timestamps
end

create_table :rails_url_shortener_ipgeos do |t|
	t.string  :ip
	t.string  :country
	t.string  :country_code
	t.string  :region
	t.string  :region_name
	t.string  :city
	t.string  :lat
	t.string  :lon
	t.string  :timezone
	t.string  :isp
	t.string  :org
	t.string  :as
	t.boolean :mobile
	t.boolean :proxy
	t.boolean :hosting
	t.timestamps
end
```

## Modelo Organization

Hay que actualizar considerar el modelo Organization y todo lo relacionado desde el primer dia. Ver [[Teams should be an MVP feature]] para mas detalles.

# Arquitectura

## Código corto para los enlaces

En una conversación con ChatGPT dijo que los códigos podrían generarse con:

- Base62 Encoding: Convierte un número en caracteres alfanuméricos (a-z, A-Z, 0-9).
- UUID + Corta longitud: Uso de identificadores únicos globales reduciendo su tamaño.

Le pregunté entre usar Base62 o SecureRandom y dijo que Base62 sería más eficiente. SecureRandom podría ser para URLs más seguras.

## Generación de código corto

El ejemplo que hizo ChatGPT fue con `SecureRandom.alphanumeric(6)`:
```ruby
def generate_short_code
	self.short_code = loop do
		random_code = SecureRandom.alphanumeric(6)
		
		break random_code unless ShortUrl.exists?(short_code: random_code)
	end
end
```

En YLL también usa SecureRandom pero pasando 8 como parámetro:
```ruby
def generate_unique_code
	self.code ||= loop do
		random_code = SecureRandom.alphanumeric(8)
		
		break random_code unless self.class.exists?(code: random_code)
	end
end
```

RailsUrlShortner hace algo más complejo:
```ruby
def key_candidate
	(0...key_length).map { RailsUrlShortener.charset[rand(RailsUrlShortener.charset.size)] }.join
end

def generate_key
	return unless key.nil?

	loop do
		# plus to the key length if after 10 attempts was not found a candidate
		self.key_length += 1 if generating_retries >= 10
		self.key = key_candidate
		self.generating_retries += 1
		break unless self.class.exists?(key: key)
	end
end
```