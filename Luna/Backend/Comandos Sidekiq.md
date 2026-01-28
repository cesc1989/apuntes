# Comandos para Revisar Colas en Sidekiq

Etiquetas: #luna_help_desk  #comanditos 

Comandos varios para poder revisar qué pasa con los jobs o colas en Sidekiq desde una consola de Rails.

## Inspeccionar un Job en específico

```ruby
Sidekiq::Queue.new.select { |job| job.klass == "MarketplaceSyncCarePlanWorker" }.each do |job|
  puts "Care Plan ID: #{job.args.first}, Enqueued at: #{Time.at(job.enqueued_at)}"
end; nil
```

Nota:
- Solo cambiar el nombre del worker.
- El `; nil` al final es para que la salida solo sean los puts.

### Ver lo que ofrece `job`

Para inspeccionar a fondo el objeto `job`:
```ruby
Sidekiq::Queue.new.select { |job| job.klass == "WiltedTreeExercisesViaWebReminderWorker" }.each do |job|
  puts job.inspect
end
```

## Estados de Colas y Procesos

Con una sola línea:
```ruby
stats = Sidekiq::Stats.new; puts "#{Time.now.strftime('%H:%M:%S')} - Queue: #{stats.enqueued} | Processing: #{Sidekiq::Workers.new.size} | Workers: #{stats.processes_size}"
```

Saca algo como:
```
15:45:36 - Queue: 1332 | Processing: 161 | Workers: 20
```

## Conteo de Jobs Programados de una Clase

```ruby
Sidekiq::ScheduledSet.new.count { |job| job.klass == "RefreshTherapistStripeTermsOfServiceAcceptanceWorker" }
```

## Conteo de Jobs Encolados de una Clase

```ruby
Sidekiq::Queue.new.count { |job| job.klass == "RefreshTherapistStripeTermsOfServiceAcceptanceWorker" }
```

## Eliminar Jobs encolados de una Clase

```ruby
Sidekiq::ScheduledSet.new.each do |job|
	if job.klass == "RefreshTherapistStripeTermsOfServiceAcceptanceWorker"
		job.delete
		puts "Deleted job scheduled at #{Time.at(job.at)}"
	end
end
```

## Obtener Jobs en Retry
```ruby
Sidekiq::RetrySet.new.select { |j| j.klass == "RefreshTherapistsStripeTermsOfServiceAcceptanceWorker" }
```