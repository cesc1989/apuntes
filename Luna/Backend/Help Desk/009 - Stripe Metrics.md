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

## Revisando la Cache

### En Development

En entorno local no hay configuración por defecto de Redis. Toca usar MemoryStore o configurar Redis.

### En Alpha

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

