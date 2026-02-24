# Configurando Bugsink como reemplazo de Sentry

La oferta de Sentry es amplia pero a la vez el servicio es muy caro. Ofrece muchas cosas pero no las necesito todas. Vi que en Pikapods está Bugsink, lo revisé y me sonó como alternativa para pasarme.

Eso hice.

## Detalles de Bugsink

Website: https://www.bugsink.com/

Es hecho por un independientes. Está escrito en Python. Funciona con el mismo SDK de Sentry así que es breve usarlo. Solo hay que cambiar el DSN.

### Recursos 🧾

Según Pikapods necesita mínimo:

- 1 cpu
- 1gb de RAM

### Costo 💰

Febrero 23, 2026: 2.73 usd/mes

## Configuración

Una vez lanzado el proyecto en Pikapods (siguiendo las [instrucciones](https://www.bugsink.com/docs/settings/) para las variables de entorno) no había mucho que hacer en la UI. Solo crear un proyecto y cambiar el DSN en el proyecto.

Quedó configurado en: https://thistle-lionfish.pikapod.net/

Los datos de acceso están en Bitwarden.

## Pruebas

Hice una prueba en local y pude apreciar esto en los logs:
```
Sentry HTTP Transport will connect to https://thistle-lionfish.pikapod.net
Initializing the Sentry background worker with 4 threads
#<StandardError: hola>
[Transport] Sending envelope with items [event] 90178c0770dc4ca6b233a107a89729e8 to Sentry
```

Se estaba conectando correctamente al server de Bugsink.

Luego en prod probé de nuevo y vi esto otro al mandar un error directo con `ExceptionLogger`:
```
ExceptionLogger.log(StandardError.new, StandardError.new("hola"))

#<StandardError: hola>

=> #<Sentry::ErrorEvent @event_id="351a80ad18d943c6b1f5a84184a0618e", @timestamp="2026-02-23T19:48:45Z", @platform=:ruby, @type="event", @sdk={"name" => "sentry.ruby", "version" => "5.26.0"}, @user={}, @extra={}, @contexts={os: {name: "Linux", version: "#1 SMP PREEMPT_DYNAMIC Tue Jul 30 15:03:21 EDT 2024", build: "6.10.2-x86_64-linode165", kernel_version: "#1 SMP PREEMPT_DYNAMIC Tue Jul 30 15:03:21 EDT 2024", machine: "x86_64"}, runtime: {name: "ruby", version: "ruby 3.4.8 (2025-12-17 revision 995b59f666) +PRISM [x86_64-linux]"}, trace: {trace_id: "cc5c58a9184d440699a2168a4821f31b", span_id: "66850dcf43ff47a2", parent_span_id: nil}}, @tags={}, @attachments=[], @fingerprint=[], @dynamic_sampling_context={"trace_id" => "cc5c58a9184d440699a2168a4821f31b", "environment" => "production", "release" => "b2bb1a7c2a2f07f94ff6f8fc0756981eef179cb2", "public_key" => "f61b9c21508e4cdd8ecc09997ce77ba3"}, @server_name="localhost", @environment="production", @release="b2bb1a7c2a2f07f94ff6f8fc0756981eef179cb2", @message="", @exception=#<Sentry::ExceptionInterface:0x000072d8265c5280 @values=[#<Sentry::SingleExceptionInterface @type="StandardError", @value="StandardError (StandardError)", @module="", @thread_id=8400, @mechanism=#<Sentry::Mechanism:0x000072d8265f7618 @type="generic", @handled=true>>]>, @threads=#<Sentry::ThreadsInterface:0x000072d82659bcc8 @id=8400, @name=nil, @current=true, @crashed=true, @stacktrace=nil>, @level="error", @breadcrumbs=#<Sentry::BreadcrumbBuffer:0x000072d8265c4e70 @buffer=[nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil, nil]>>
```

Y pude ver ambos errores:
![[bugsink.initial.setup.png]]