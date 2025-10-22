# 010 - Tamper Detection Evidence for PCI DSS Compliance

Etiquetas: #luna_help_desk 

Caso EDG-2888

Cosas de seguridad. Fausto pide verificar que hay una configuración que protege sobre suplantación de identidad al hacer peticiones a Stripe.

> we must demonstrate that a mechanism is in place to detect unauthorized modifications to the payment page and its HTTP headers, as received by the consumer’s browser.

Según Alexis ya esto está cubierto por lo que hay en `config/initializers/content_security_policy.rb`. Me queda de trabajo comprobar que funciona.

# Configuración de Content Security Policy

Explicación en las guías de Rails para la versión 7.1: https://guides.rubyonrails.org/v7.1/security.html#content-security-policy-header

Fausto dice que:
> I suggest adding SRI and CSP with the report-to directive

Esa directiva está soportada con la función `report_uri` de Rails.

Así está configurada a la fecha:
```ruby
# Specify URI for violation reports
if Luna.env.local? || Luna.env.test?
	# If you are using webpack-dev-server then specify webpack-dev-server host
	policy.connect_src :self, Luna.env_protocol, "http://localhost:3035", "ws://localhost:3035", "https://*.stripe.com", "https://maps.googleapis.com"
	policy.report_uri "/csp-violation-report-endpoint"
else
	policy.connect_src :self, Luna.env_protocol, "https://sentry.io", "https://*.stripe.com", "https://maps.googleapis.com"
	policy.report_uri ENV["SENTRY_SECURITY_HEADER_URL"]
end
```

Para local se envía la información al romper la política de seguridad al endpoint `"/csp-violation-report-endpoint"`. Sin embargo, no hay controlador para ello porque en local no es necesario.

> [!Note]
> Voy a crear uno con Claudio para las pruebas locales.

Para producción (alpha y omega) se envía el fallo a `ENV["SENTRY_SECURITY_HEADER_URL"]`.

# Probar Suplantar en Página de Pagos Stripe

## Cabecera HTTP de Respuesta

La forma más simple es revisar la cabecera `Content-Security-Policy-Report-Only` en la respuesta al cargar el admin.

Esto puedo ver en local:
```
default-src 'self' http:; font-src 'self' http: data:; frame-src 'self' http: https://*.stripe.com; img-src 'self' http: data: https://*.stripe.com; object-src 'none'; script-src 'self' http: data: 'unsafe-inline' 'unsafe-eval' https://luna-live-infra-assets.s3.us-west-2.amazonaws.com https://*.stripe.com https://cdn.ingest-lr.com; style-src 'self' http: data: 'unsafe-inline' https://luna-live-infra-assets.s3.us-west-2.amazonaws.com https://use.fontawesome.com; worker-src 'self' blob:; connect-src 'self' http: http://localhost:3035 ws://localhost:3035 https://*.stripe.com https://maps.googleapis.com; report-uri /csp-violation-report-endpoint
```

Ahí al final aparece `report-uri /csp-violation-report-endpoint`.

## Controlador local

En local hay que crear una ruta y controlador para el endpoint definido (`/csp-violation-report-endpoint`).

Ruta:
```ruby
post "/csp-violation-report-endpoint", to: "csp_violations#report"
```

Controlador en `app/controllers/csp_violations_controller.rb`:
```ruby
# frozen_string_literal: true

# Handles Content Security Policy violation reports for local development
class CspViolationsController < ApplicationController
  skip_before_action :verify_authenticity_token

  def report
    Rails.logger.info("📥 CSP Violation Report Received - Content-Type: #{request.content_type}")

    body = request.body.read
    Rails.logger.info("📦 Raw Body: #{body}")

    violation_report = JSON.parse(body)
    Rails.logger.warn("🚨 CSP Violation Detected:")
    Rails.logger.warn(JSON.pretty_generate(violation_report))

    head :ok
  rescue JSON::ParserError => e
    Rails.logger.error("❌ Failed to parse CSP violation report: #{e.message}")
    Rails.logger.error("Raw body was: #{body}")
    head :bad_request
  rescue StandardError => e
    Rails.logger.error("❌ Unexpected error: #{e.class} - #{e.message}")
    Rails.logger.error(e.backtrace.join("\n"))
    head :internal_server_error
  end
end
```

> [!Important]
> Este controlador no hace nada. Solo sirve para demostrar que el endpoint puede recibir información cuando se rompe una política.

## Prueba en local desde dev console

Claudio me dijo que puedo probar con este script. Lo pego en la dev console:
```js
var object = document.createElement('object');
object.data = 'https://evil.com/file.pdf';
object.type = 'application/pdf';
document.body.appendChild(object);
console.log('Attempted to add unauthorized object');
```

Y puedo ver esto en los logs:
```
Started POST "/csp-violation-report-endpoint" for ::1 at 2025-10-22 17:16:17 -0500
Processing by CspViolationsController#report as */*

📥 CSP Violation Report Received - Content-Type: application/csp-report
📦 Raw Body: {"csp-report":{"document-uri":"http://localhost:3000/admin/welcome","referrer":"","violated-directive":"object-src","effective-directive":"object-src","original-policy":"default-src 'self' http:; font-src 'self' http: data:; frame-src 'self' http: https://*.stripe.com; img-src 'self' http: data: https://*.stripe.com; object-src 'none'; script-src 'self' http: data: 'unsafe-inline' 'unsafe-eval' https://luna-live-infra-assets.s3.us-west-2.amazonaws.com https://*.stripe.com https://cdn.ingest-lr.com; style-src 'self' http: data: 'unsafe-inline' https://luna-live-infra-assets.s3.us-west-2.amazonaws.com https://use.fontawesome.com; worker-src 'self' blob:; connect-src 'self' http: http://localhost:3035 ws://localhost:3035 https://*.stripe.com https://maps.googleapis.com; report-uri /csp-violation-report-endpoint","disposition":"report","blocked-uri":"https://evil.com","status-code":200,"script-sample":""}}

🚨 CSP Violation Detected:
{
  "csp-report": {
    "document-uri": "http://localhost:3000/admin/welcome",
    "referrer": "",
    "violated-directive": "object-src",
    "effective-directive": "object-src",
    "original-policy": "default-src 'self' http:; font-src 'self' http: data:; frame-src 'self' http: https://*.stripe.com; img-src 'self' http: data: https://*.stripe.com; object-src 'none'; script-src 'self' http: data: 'unsafe-inline' 'unsafe-eval' https://luna-live-infra-assets.s3.us-west-2.amazonaws.com https://*.stripe.com https://cdn.ingest-lr.com; style-src 'self' http: data: 'unsafe-inline' https://luna-live-infra-assets.s3.us-west-2.amazonaws.com https://use.fontawesome.com; worker-src 'self' blob:; connect-src 'self' http: http://localhost:3035 ws://localhost:3035 https://*.stripe.com https://maps.googleapis.com; report-uri /csp-violation-report-endpoint",
    "disposition": "report",
    "blocked-uri": "https://evil.com",
    "status-code": 200,
    "script-sample": ""
  }
}

Completed 200 OK in 2ms (ActiveRecord: 0.0ms | Allocations: 797)
```

## Pruebas en Alpha y Omega

Pude usar el mismo script JS y comprobar que en Sentry se recibe el error de la violación de la política.