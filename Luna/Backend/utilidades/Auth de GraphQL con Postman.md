# Auth de GraphQL con Postman

¿Cómo autentico las peticiones graphql desde Postman?

Necesito encontrar una cuenta de Therapist:
```ruby
acc = Therapist.first.account
```

Con esa account invoco al método `create_new_auth_token`:
```ruby
token = acc.create_new_auth_token
```

Ese método devuelve un hash con las credenciales para hacer peticiones. Sin embargo, es un Hash de Ruby y no lo puedo pegar tal cual en las cabeceras de Postman.

Para ayudarme con eso tengo la función `headers_from_ruby_hash`. La clave de esta función es pasarle el hash en una sola línea, sin espacios para evitar problemas.

Entonces para eso generamos el token usando `p` desde la consola:
```ruby
p acc.create_new_auth_token
{"access-token"=>"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIwMDAwMjZlOS1kNDExLTQ1ZDQtYTExOS1iMzQ2ZTBiYWYxYTAiLCJpYXQiOjE3NDc2ODA3ODN9.nXJ0-_TflXCYrgxnw-ulyowWGBDsQXwy3I8N6y4p3g0", "token-type"=>"Bearer", "client"=>"_GRP14pt7412EatY_B-Jxw", "expiry"=>"1807680783", "uid"=>"demo+1713926474971@koombea.com", "Authorization"=>"Bearer eyJhY2Nlc3MtdG9rZW4iOiJleUpoYkdjaU9pSklVekkxTmlKOS5leUp6ZFdJaU9pSXdNREF3TWpabE9TMWtOREV4TFRRMVpEUXRZVEV4T1MxaU16UTJaVEJpWVdZeFlUQWlMQ0pwWVhRaU9qRTNORGMyT0RBM09ETjkublhKMC1fVGZsWENZcmd4bnctdWx5b3dXR0JEc1FYd3kzSThONnk0cDNnMCIsInRva2VuLXR5cGUiOiJCZWFyZXIiLCJjbGllbnQiOiJfR1JQMTRwdDc0MTJFYXRZX0ItSnh3IiwiZXhwaXJ5IjoiMTgwNzY4MDc4MyIsInVpZCI6ImRlbW8rMTcxMzkyNjQ3NDk3MUBrb29tYmVhLmNvbSJ9"}
```

Ese hash lo copiamos y así lo usamos en una terminal:
```bash
ruby_hash='{"access-token"=>"eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJjYTkyZDQwOC1kZjdlLTRiYWUtOTJhMS01ODc0Y2VhYmJlZDQiLCJpYXQiOjE3NDM1MTYwMTh9.dgtHGQEBVnKL6Vok1-jkf3X1okoWnFQmoTBvjXpnhRA", "token-type"=>"Bearer", "client"=>"kPaUWyE92m9eq_q7J20lWg", "expiry"=>"1803516018", "uid"=>"steve+nt@getluna.com", "Authorization"=>"Bearer eyJhY2Nlc3MtdG9rZW4iOiJleUpoYkdjaU9pSklVekkxTmlKOS5leUp6ZFdJaU9pSmpZVGt5WkRRd09DMWtaamRsTFRSaVlXVXRPVEpoTVMwMU9EYzBZMlZoWW1KbFpEUWlMQ0pwWVhRaU9qRTNORE0xTVRZd01UaDkuZGd0SEdRRUJWbktMNlZvazEtamtmM1gxb2tvV25GUW1vVEJ2alhwbmhSQSIsInRva2VuLXR5cGUiOiJCZWFyZXIiLCJjbGllbnQiOiJrUGFVV3lFOTJtOWVxX3E3SjIwbFdnIiwiZXhwaXJ5IjoiMTgwMzUxNjAxOCIsInVpZCI6InN0ZXZlK250QGdldGx1bmEuY29tIn0="}'
```

> Nota las comillas simples.

Y así invoco la función:
```bash
headers_from_ruby_hash "$ruby_hash"
```

> Nota las comillas dobles.

Si todo está en orden, habrá una respuesta como esta:
```bash
access-token: eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIwMDAwMjZlOS1kNDExLTQ1ZDQtYTExOS1iMzQ2ZTBiYWYxYTAiLCJpYXQiOjE3NDc2ODA3ODN9.nXJ0-_TflXCYrgxnw-ulyowWGBDsQXwy3I8N6y4p3g0
token-type: Bearer
client: _GRP14pt7412EatY_B-Jxw
expiry: 1807680783
uid: demo+1713926474971@koombea.com
Authorization: Bearer eyJhY2Nlc3MtdG9rZW4iOiJleUpoYkdjaU9pSklVekkxTmlKOS5leUp6ZFdJaU9pSXdNREF3TWpabE9TMWtOREV4TFRRMVpEUXRZVEV4T1MxaU16UTJaVEJpWVdZeFlUQWlMQ0pwWVhRaU9qRTNORGMyT0RBM09ETjkublhKMC1fVGZsWENZcmd4bnctdWx5b3dXR0JEc1FYd3kzSThONnk0cDNnMCIsInRva2VuLXR5cGUiOiJCZWFyZXIiLCJjbGllbnQiOiJfR1JQMTRwdDc0MTJFYXRZX0ItSnh3IiwiZXhwaXJ5IjoiMTgwNzY4MDc4MyIsInVpZCI6ImRlbW8rMTcxMzkyNjQ3NDk3MUBrb29tYmVhLmNvbSJ9
```

> [!Note]
> Cuando se copie a Postman el valor de la cabecera _Authorization_ podría tener salto de línea. Cuidado con eso.