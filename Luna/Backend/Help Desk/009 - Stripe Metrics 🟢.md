# 009 - Stripe Metrics

Etiquetas: #luna_help_desk 

Caso EDG-824

Hay que generar métricas de uso de Stripe para poder configurar alarmas cuando haya un alto bloqueo de las peticiones hacía Stripe.

# Solución

Incrementar una cache cada que se haga un retry en un llamado a la API de Stripe y también cuando fallen los tres retries configurados.

Esta es la solución:
```ruby
config.contexts[:stripe] = {
	on: [Stripe::RateLimitError, Net::ReadTimeout, Faraday::TimeoutError],
	sleep_method:
		proc do
			# Randomize to prevent a thundering herd.
			seconds = Kernel.rand(1.0..5.0)
			sleep(seconds)
		end,
	tries: 3,
	log_method:
		proc do |retries, exception|
			MetricsService.increment_stripe_api_call_attempts_total

			# retries + 1 >= tries means we're about to give up after this attempt
			if retries + 1 >= 3
				MetricsService.increment_stripe_api_call_failures_total

				Rails.logger.error(
					"Stripe API call failed after #{retries + 1} attempts: #{exception.class} - #{exception.message}"
				)
			end
		end
}
```

El bloque `log_method`. La clave de este bloque es que solo se ejecuta en las fallas y tiene acceso al número de retries.

## Implementación con Monkey patch de Alexis

Alexis agregó este patch al inicializador:
```ruby
module RetryableGiveUpCallback
  def retryable(options = {}, &block)
    give_up_cb = options.delete(:give_up_cb)
    last_retries = 0

    super(options) do |retries, retry_exception|
      last_retries = retries
      block.call(retries, retry_exception)
    end
  rescue *Array(options.fetch(:on, StandardError)) => e
    # If we get here, retries are exhausted
    give_up_cb&.call(e, last_retries)
    raise
  end
end
```

Para luego actualizar el context de esta forma:
```ruby
config.contexts[:stripe] = {
	on: [Stripe::RateLimitError, Net::ReadTimeout, Faraday::TimeoutError],
	sleep_method:
		proc do
			# Randomize to prevent a thundering herd.
			seconds = Kernel.rand(1.0..5.0)
			sleep(seconds)
		end,
	tries: 3,
	ensure:
		proc do |retries|
			puts "desde ensure"
			MetricsService.increment_stripe_api_call_attempts_total(retries + 1)
		end,
	exception_cb:
		proc do |_retry_exception|
			puts "desde exception_cb"
			MetricsService.increment_stripe_api_call_retries_total
		end,
	give_up_cb:
		proc do |_retry_exception, _retries|
			puts "desde give_up_cb"
			MetricsService.increment_stripe_api_call_failures_total
		end
}
```

Al probar el script retorna esto:
```
Initial failures: 0
Initial attempts: 0
Initial retries: 0
desde exception_cb
desde exception_cb
desde ensure
desde give_up_cb
Expected failure: Test rate limit
After failure - Attempts: 3
After failure - Failures: 1
After failure - Retries: 2
```

¿Qué significa esto ahora comparado con la versión anterior?
```
Initial failures: 0
Initial attempts: 0

Llamado 1
Llamado 2

Stripe API call failed after 3 attempts: Stripe::RateLimitError - Test rate limit
Expected failure: Test rate limit

After failure - Attempts: 2
After failure - Failures: 1
```

> [!Important]
> Solución:
> La diferencia está en que la implementación con el monkey patch permite calcular el momento justo en que Retryable deja de intentar. Es decir no hay forma alguna de recuperarse del error. O sea que el proceso falló definitivamente.
>
> Retryable funciona intentando la cantidad indicada por el atributo `tries`. Así que para un `tries = 3` Retryable lo que hace es contabilizar dos intentos después del primero obligatorio.
> .
> Con el monkey patch podemos tener todos los estados: la cantidad de intentos (3), los reintentos (2) y el fallo definitivo e irrecuperable (1).

# Revisando la Cache

## En Development

En entorno local no hay configuración por defecto de Redis. Toca usar MemoryStore o configurar Redis.

Activé la cache con `bundle exec rails dev:cache` y pude probar.

Arroja esto al correr en rails console:
```
Initial failures: 0
Initial attempts: 0
Stripe API call failed after 3 attempts: Stripe::RateLimitError - Test rate limit
Expected failure: Test rate limit
After failure - Attempts: 2
After failure - Failures: 1
Failure rate: 50.0%
```

## En Alpha

Para revisar las llaves de métricas en alpha:
```ruby
Rails.cache.redis.keys("*metric*")
```

Responde:
```ruby
["luxe_cache:metric/gql_cache_hits_total",
 "luxe_cache:metric/candid_sync_eligible_appointment_total/evaluated_at",
 "luxe_cache:metric/inactive_appointment_count",
 "luxe_cache:metric/candid_sync_eligible_appointment_total",
 "luxe_cache:metric/signed_appointment_count",
 "luxe_cache:metric/signed_appointment_count/evaluated_at",
 "luxe_cache:metric/gql_cache_requests_total",
 "luxe_cache:metric/inactive_appointment_count/evaluated_at"]
```

# Yabeda

Las métricas por si solas no hacen nada. Para poder hacerlas efectivas se necesitan usar en alarmas o tableros. Este proyecto usa Yabeda para generar datos que puede leer Prometheus.

Así está configurado en el `config/routes.rb`:
```ruby
  yabeda_app = Rack::Builder.new do
    use BearerAuth do |token|
      token == ENV.fetch("YABEDA_AUTH_TOKEN")
    end
    run Yabeda::Prometheus::Exporter
  end
  mount yabeda_app, at: "/metrics"
```

La ruta `/metrics` no tiene vista sino que Yabeda responde con una estructura que puede entender Prometheus.

## Prueba de Yabeda en Local

En local:
```bash
curl -i http://localhost:3000/metrics -H "Authorization: Bearer yabeda_auth"
```

Respuesta con las keys de las métricas de Stripe:
```bash
# TYPE edge_stripe_api_call_failures_total counter
# HELP edge_stripe_api_call_failures_total Total Stripe API calls that failed after exhausting all retries
edge_stripe_api_call_failures_total 0.0
# TYPE edge_stripe_api_call_attempts_total counter
# HELP edge_stripe_api_call_attempts_total Total Stripe API call attempts (including retries)
edge_stripe_api_call_attempts_total 0.0
```

## Prueba de Yabeda en Alpha

Para probar en alpha se puede usar esta petición:
```bash
curl -i https://api2.alpha.getluna.com/metrics -H "Authorization: Bearer 327e56ec-74ee-4712-978c-feb0c7a60832"
```

Responde algo como esto post despliegue:
```bash
# TYPE edge_stripe_api_call_attempts_total counter
# HELP edge_stripe_api_call_attempts_total Total Stripe API call attempts (including retries)
edge_stripe_api_call_attempts_total 6.0
# TYPE edge_stripe_api_call_retries_total counter
# HELP edge_stripe_api_call_retries_total Total Stripe API call retries performed
edge_stripe_api_call_retries_total 4.0
# TYPE edge_stripe_api_call_failures_total counter
# HELP edge_stripe_api_call_failures_total Total Stripe API calls that failed after exhausting all retries
edge_stripe_api_call_failures_total 2.0
```

# Script para Pruebas simples

Script para prueba desde rails console:
```ruby
# 1. Initialize counters to 0 in Redis (first time setup)
MetricsService.write(:stripe_api_call_failures_total, 0)
MetricsService.write(:stripe_api_call_attempts_total, 0)
MetricsService.write(:stripe_api_call_retries_total, 0)

# 2. Verify counters are initialized
metrics = MetricsService.new
puts "Initial failures: #{metrics.stripe_api_call_failures_total}"
puts "Initial attempts: #{metrics.stripe_api_call_attempts_total}"
puts "Initial retries: #{metrics.stripe_api_call_retries_total}"

# 3. Simulate API failure
begin
  Retryable.with_context(:stripe) do
    raise Stripe::RateLimitError.new("Test rate limit")
  end
rescue Stripe::RateLimitError => e
  puts "Expected failure: #{e.message}"
end

# 4. Create NEW instance to read fresh values from Redis
metrics = MetricsService.new
puts "After failure - Attempts: #{metrics.stripe_api_call_attempts_total}"  # Should be 3
puts "After failure - Failures: #{metrics.stripe_api_call_failures_total}"  # Should be 1
puts "After failure - Retries: #{metrics.stripe_api_call_retries_total}"  # Should be 3
```