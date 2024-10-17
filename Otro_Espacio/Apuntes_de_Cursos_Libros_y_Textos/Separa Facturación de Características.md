# Separate Billing from Entitlements

[Artículo original](https://arnon.dk/why-you-should-separate-your-billing-from-entitlement/)

El título hace referencia a que en una plataforma se deben separar la facturación (cobro de dinero a los usuarios) de las características a los que ellos tienen acceso.

Por ejemplo, en un proyecto que trabajé, los usuarios accedían a las características del programa según una bandera en la BD que se asignaba según el plan que estaban pagando. El artículo aboga por un sistema separado que pinta mucho mejor.

## ¿Qué es un _entitlement_?

En términos de web apps: básicamente un derecho. Algo a lo que se tiene _derecho_ por haber pagado una mensualidad.

> An entitlement is a customer’s access to a specific feature or product, within a given plan.

El gráfico (tomado del artículo), explica como se separan los _derechos_ de la facturación.!

![[entitlements_and_billing.png]]

Importante:

3 - _a set of entitlements that make up this plan or tier. These would commonly be called features, products, or services._

## ¿Cómo ayudan?

> one day you want to modify your basic level to remove a feature that costs too much to be valuable. Now ==you must go through all places in your codebase and modify these hardcoded checks==.

> You are faced with either force-upgrading them, or making another plan level that includes this feature, effectively “grandfathering” them into a plan that can’t be selected.

Algo así se alcanzo a ver en Bucket. Había usuarios del plan de 1 dólar y había los usuarios normales.

> With an entitlement system, you manage each feature’s business logic in one central system, instead of in your applications.

![[entitlement_system.png]]

## Beneficios de un sistema de "Derechos"

Lanzar características sin un nuevo despliegue de código.

Planear lanzamientos programando el _entitlement_ en una fecha en específica. Reemplazando Feature Flags.

Se pueden cambiar las características que un _entitlement_ da acceso en minutos.