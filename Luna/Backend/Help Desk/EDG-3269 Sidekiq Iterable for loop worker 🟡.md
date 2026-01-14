# EDG-3269 - Usa Sidekiq Iterable para worker que itera en una colección ActiveRecord

Siguiendo con el tema de Sidekiq demorando para tomar los workers encolados descrito en [[024 - QA Forms Demoran en Crearse 🟡]].

En el standup del Martes comenté lo que seguía y Jeff dijo que Daulyn estaba haciendo algo similar. Hablé con Daulyn y resulta que por aquí es que puede ser el problema.

## Worker Problemático

El tema es el worker `RefreshTherapistsStripeTermsOfServiceAcceptanceWorker`. Este se ejecuta de manera diaria a las 1am UTC.

En el worker está la siguiente query:
```ruby
Therapist.activated_or_pending_activation
```

Para Alpha, esa query retorna 16K registros. En Omega son solo 2K. El problema no es solo eso sino que ese worker encola a `RefreshTherapistStripeTermsOfServiceAcceptanceWorker` por tantos registros retorne la consulta.

Así que tenemos una situación donde el worker padre encola al menos 16K workers adicionales. Eso es inviable.

Cuando voy a Sidekiq web en Alpha, en la pestaña Programadas hay muchas páginas y todas tienen el mismo worker en cola:
![[same.worker.multiple.schedules.png]]

## Solución

La solución está en usar `Sidekiq::Iterable` en vez de `Sidekiq::Job`. Esto es una característica disponible desde Sidekiq 7.3.

Docs:

- [Iteration and Sidekiq 7.3.0](https://www.mikeperham.com/2024/07/03/iteration-and-sidekiq-7.3.0/)
- [En la Wiki de Sidekiq](https://github.com/sidekiq/sidekiq/wiki/Iteration)

## Pruebas

Antes de probar en Alpha quiero saber cómo revisar, desde la consola, la cantidad de workers encolados de `RefreshTherapistStripeTermsOfServiceAcceptanceWorker`.

### ¿Cuántos jobs de la clase están encolados?

Lo pude saber con este comando:
```ruby
Sidekiq::ScheduledSet.new.count { |job| job.klass == "RefreshTherapistStripeTermsOfServiceAcceptanceWorker" }
```

Retornó 12_432.

Hay que limpiarlos antes de mezclar los cambios a Alpha.

### Borrar jobs encolados de la clase

Se tienen que borrar porque el job singular se eliminó.

Se hace así:
```ruby
Sidekiq::ScheduledSet.new.each do |job|
	if job.klass == "RefreshTherapistStripeTermsOfServiceAcceptanceWorker"
		job.delete
		puts "Deleted job scheduled at #{Time.at(job.at)}"
	end
end
```

