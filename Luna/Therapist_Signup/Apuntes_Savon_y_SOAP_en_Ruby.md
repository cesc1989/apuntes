# Apuntes Savon y SOAP en Ruby

## Enlaces

- Página de [Savon](https://www.savonrb.com/)
- En [Stack Overflow](https://stackoverflow.com/questions/40273/whats-the-best-way-to-use-soap-with-ruby)
- [Tutorial en Español](https://vimeo.com/240413119)

## Detalles

- Hay que leer el [WSDL](https://es.wikipedia.org/wiki/WSDL) (web service description language): este dice los nombres de atributos y nombres de endpoints/servicios.
- Savon hace transformación de atributos/endpoints de PascalCase a snake_case.
- El man del tuto anterior sacó eso de un [railscasts](http://railscasts.com/episodes/290-soap-with-savon?autoplay=true) de Ryan Bates
- Savon [solo recibe actualizaciones de errores o mantenimiento](https://github.com/savonrb/savon/issues/904#issuecomment-564581970) para la versión 2

## SOAP web services para pruebas

En este [comentario](https://stackoverflow.com/questions/311654/public-free-web-services-for-testing-soap-client#comment123225530_311661) enlazan a un [postman](https://documenter.getpostman.com/view/8854915/Szf26WHn#4ba276a7-b12c-4943-ae23-c6031cc48248) con diferentes endpoints.

# Probando Savon

## Cliente

El cliente se puede crear usando un WSDL. Esta es la mejor forma para no tener problemas.
```ruby
client = Savon.client(wsdl: "https://www.dataaccess.com/webservicesserver/NumberConversion.wso?wsdl")
    
ap client.operations
[
	[0] :number_to_words,
	[1] :number_to_dollars
]
```
  

Con la forma de namespace y endpoint no funciona `client.operations`
```ruby
client = Savon.client(
  endpoint: 'https://www.dataaccess.com/webservicesserver/NumberConversion.wso',
  namespace: 'http://www.dataaccess.com/webservicesserver/'
)
ap client.operations

/savon-2.14.0/lib/savon/client.rb:89:in raise_missing_wsdl_error!: Unable to inspect the service without a WSDL document. (RuntimeError)
```

Ni tampoco petición a una operación:
```ruby
client = Savon.client(
  endpoint: "https://www.dataaccess.com/webservicesserver/NumberConversion.wso",
  namespace: "http://www.dataaccess.com/webservicesserver/"
)
    
payload = { ubiNum: 100 }
r = client.call(:number_to_words, message: payload)

savon-2.14.0/lib/savon/response.rb:132:in `raise_soap_and_http_errors!': (soap:Server) Error processing request: No such method: numberToWords (Savon::SOAPFault)
```

Si lo defino así:
```ruby
client = Savon.client do
  endpoint 'https://www.dataaccess.com/webservicesserver/NumberConversion.wso'
  namespace 'http://www.dataaccess.com/webservicesserver/'
  convert_request_keys_to :camelcase
end
    
payload = { ubiNum: 100 }
r = client.call(:number_to_words, message: payload)
```

Ya no explota por el nombre de la operación pero sí por el parámetro:
```
savon-2.14.0/lib/savon/response.rb:132:in `raise_soap_and_http_errors!': (soap:Server) Error processing request: Unexpected parameter 'UbiNum' (Savon::SOAPFault)
```

No debe ser `UbiNum` sino como está en el payload. Para solucionarlo hay que enviar la llave como string.
```ruby
client = Savon.client do
  endpoint 'https://www.dataaccess.com/webservicesserver/NumberConversion.wso'
  namespace 'http://www.dataaccess.com/webservicesserver/'
  convert_request_keys_to :camelcase
end

payload = { "ubiNum" => 100 }
r = client.call(:number_to_words, message: payload)

ap r.success?
true

ap r
{
	:number_to_words_response => {
		:number_to_words_result => "one hundred ",
					:"@xmlns:m" => "http://www.dataaccess.com/webservicesserver/"
	}
}
```


## Peticiones

Para hacer una petición a una operación SOAP que recibe este cuerpo XML:
```xml
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
  <soap:Body>
	<NumberToWords xmlns="http://www.dataaccess.com/webservicesserver/">
	  <ubiNum>500</ubiNum>
	</NumberToWords>
  </soap:Body>
</soap:Envelope>
```

En Savon sería así:
```ruby
 client = Savon.client(wsdl: "https://www.dataaccess.com/webservicesserver/NumberConversion.wso?wsdl")
    
payload = { "ubiNum" => 100 }
r = client.call(:number_to_words, message: payload)
```

Donde la operación está dada por la etiqueta: `NumberToWords` y los parámetros están dentro de la etiqueta, en este caso, `ubiNum`.

Savon convierte todas las operaciones a símbolos y también se encarga de convertir los símbolos a etiquetas XML. Por eso la petición se puede hacer así:
```ruby
client.call(:number_to_words, message: { "ubiNum" => 100 })
```

Nota que usé el cuerpo `ubiNum` en string pero también pude haber hecho:
```ruby
client.call(:number_to_words, message: { ubiNum: 100 })
```


## Respuesta

Savon provee métodos para inspeccionar la respuesta a la operación:
```ruby
r.header # si hay cabeceras, devuelve un hash
    
r.body # el cuerpo lo devuelve como hash
{
	:number_to_words_response => {
		:number_to_words_result => "one hundred ",
					:"@xmlns:m" => "http://www.dataaccess.com/webservicesserver/"
	}
}

r.success? # si es un 2xx
true
```


## Modelo Savon

Según los [docs](https://www.savonrb.com/version2/model.html) se puede extender una clase para darle los métodos de Savon. Esto puede ser conveniente pero se pierden algunas ventajas de la forma directa de uso.

Defino un modelo
```ruby
# app/lib/symplr.rb
class Symplr
  extend Savon::Model

  client(wsdl: "https://www.dataaccess.com/webservicesserver/NumberConversion.wso?wsdl")

  operations(:number_to_words)
end
```

Y funciona:
```ruby
c = Symplr.new
r = c.number_to_words(message: { ubiNum: 20 })
ap r
{
	:number_to_words_response => {
		:number_to_words_result => "twenty ",
					:"@xmlns:m" => "http://www.dataaccess.com/webservicesserver/"
	}
}
```

Pero toca sí o sí definir el macro `operations` en la clase. Sin eso definido no se pueden invocar las operaciones del web service:
```ruby
# app/lib/symplr.rb
class Symplr
  extend Savon::Model

  client(wsdl: "https://www.dataaccess.com/webservicesserver/NumberConversion.wso?wsdl")
end

c = Symplr.new
r = c.number_to_words(message: { ubiNum: 20 })

undefined method `number_to_words' for #<Symplr:0x000000014aefdc60> (NoMethodError)
```

Además que tampoco se obtiene la lista de operaciones disponibles:
```ruby
ap Symplr.operations
[]
```

