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