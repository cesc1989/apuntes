# Guia de Timeouts en Ruby

Web: https://github.com/ankane/the-ultimate-guide-to-ruby-timeouts

Introducción:
> An unresponsive service can be worse than a down one. It can tie up your entire system if not handled properly. **All network requests should have a timeout.**

Dicen que:
> You should [avoid Ruby’s `Timeout` module](https://www.mikeperham.com/2015/05/08/timeout-rubys-most-dangerous-api/).

## Tipos de Timeout

- **connect (or open)** - time to open the connection
- **read (or receive)** - time to receive data after connected
- **write (or send)** - time to send data after connected
- **checkout** - time to checkout a connection from the pool
- **statement** - time to execute a database statement
- **lock (or acquisition)** - time to acquire a lock
- **request (or service)** - time to process a request
- **wait** - time to start processing a queued request
- **command** - time to run a command
- **solve** - time to solve an optimization problem


## Statement Timeouts

La guía los enfoca en los motores de bases de datos.

### En PostgreSQL

En `config/database.yml`:

```yaml
production:
  variables:
    statement_timeout: 5s # or ms, min, etc
```

Dice que se puede probar así:
```
SELECT pg_sleep(6);
```

Cuando se corran migraciones hay que subirle. Entonces la configuración puede leer el valor de una ENV:
```yaml
production:
  variables:
    statement_timeout: <%= ENV["STATEMENT_TIMEOUT"] || "5s" %>
```

Y usarse así:
```
STATEMENT_TIMEOUT=90s rails db:migrate
```

## Librerías

Cómo configurar timeout en gemas que he usado.

### net/http

```ruby
Net::HTTP.start(host, port, open_timeout: 1, read_timeout: 1, write_timeout: 1) do
  # ...
end
```

o de esta otra forma:
```ruby
http = Net::HTTP.new(host, port)
http.open_timeout = 1
http.read_timeout = 1
http.write_timeout = 1
```

Levanta las excepciones:
- `Net::OpenTimeout` on connect timeout
- `Net::ReadTimeout` on read timeout
- `Net::WriteTimeout` on write timeout

### activerecord

Cuando es con postgresql, en `config/database.yml`:
```yaml
production:
  connect_timeout: 1
  checkout_timeout: 1
```

Levanta las excepciones:
- `ActiveRecord::ConnectionNotEstablished` on connect and read timeouts
- `ActiveRecord::ConnectionTimeoutError` on checkout timeout

### Faraday

Así:
```ruby
Faraday.get(url) do |req|
  req.options.open_timeout = 1
  req.options.timeout = 1
end
```

Levanta:
- `Faraday::ConnectionFailed` on connect timeout
- `Faraday::TimeoutError` on read timeout

### http.rb

```ruby
HTTP.timeout(connect: 1, read: 1, write: 1).get(url)
```

Levanta:
Raises

- `HTTP::ConnectTimeoutError` on connect timeout
- `HTTP::TimeoutError` on read timeout

### httparty

Así:
```ruby
HTTParty.get(url, timeout: 1)
```

O así:
```ruby
class Resource
  include HTTParty

  default_timeout 1
  # or
  open_timeout 1
  read_timeout 1
  write_timeout 1
end
```

Levanta:
- `Net::OpenTimeout` on connect timeout
- `Net::ReadTimeout` on read timeout

### aws-sdk

Desde el initializer:
```ruby
Aws.config = {
  http_open_timeout: 1,
  http_read_timeout: 1
}
```

o desde instancia:
```ruby
Aws::S3::Client.new(
  http_open_timeout: 1,
  http_read_timeout: 1
)
```

Levanta: `Seahorse::Client::NetworkingError`

### sentry-ruby

```ruby
Sentry.init do |config|
  config.transport.open_timeout = 1
  config.transport.timeout = 1
end
```

Raises `Sentry::ExternalError` in [some cases](https://github.com/getsentry/sentry-ruby/issues/1290)

### actionmailer

```ruby
ActionMailer::Base.smtp_settings = {
  open_timeout: 1,
  read_timeout: 1
}
```

Levanta:
- `Net::OpenTimeout` on connect timeout
- `Net::ReadTimeout` on read timeout

## Rescatar Excepciones de Timeouts

En vez de hacer:
```ruby
rescue Net::OpenTimeout, Net::ReadTimeout
```

haz:
```ruby
rescue Timeout::Error
```

### Tips

- `Timeout::Error` for both `Net::OpenTimeout` and `Net::ReadTimeout`
- `Faraday::ClientError` for both `Faraday::ConnectionFailed` and `Faraday::TimeoutError`
- `HTTPClient::TimeoutError` for both `HTTPClient::ConnectTimeoutError` and `HTTPClient::ReceiveTimeoutError`
- `Redis::BaseConnectionError` for both `Redis::CannotConnectError` and `Redis::TimeoutError`
- `Rack::Timeout::Error` for both `Rack::Timeout::RequestTimeoutError` and `Rack::Timeout::RequestExpiryError`
- `RestClient::Exceptions::Timeout` for both `RestClient::Exceptions::OpenTimeout` and `RestClient::Exceptions::ReadTimeout`