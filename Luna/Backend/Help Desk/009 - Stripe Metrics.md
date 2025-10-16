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
	exception_cb:
		proc do |_exception, try_number|
			MetricsService.increment_stripe_api_call_attempts_total

			MetricsService.increment_stripe_api_call_failures_total if try_number == 3
		end
}
```

El bloque `exception_cb`.

## Revisando la Cache