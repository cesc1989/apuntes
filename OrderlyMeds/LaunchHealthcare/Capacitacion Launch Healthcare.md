# Apuntes Capacitación Plataforma Launch Healthcare

> [!Important]
> La claves de esta plataforma:
> 
> > _Anyone attending patients can send patients to a send patients to a website to buy GLP1_.
>
> > _Start a new brand/client is a configuration in the Admin side of the platform_.
>
> > _==Ahora OrderlyMeds va es atender a Empresas. Pasa de ser B2C a ser B2B==_.
>
>> La primera línea de verificación es el Admin Portal. Revisar el Catalog y las Variantes.
>
>> 

## Personas de Contacto

- Jeff Carroll
	- Power User
	- Encargado y más cercano al Sistema / Negocio
- Varun Nehra
	- Arquitecto y Desarrollador


## Nombres que ha tenido la plataforma

Nombres que ha tenido. El nombre definitivo es **Launch Healthcare**.

- Health as a Service
- Health Grid
- Launch Healthcare

## Nomenclatura 📚

> [!Important]
> A pesar de que ellos dicen la palabra "Tenant" este sistema no es multitenant en el sentido de una base de datos por cada cliente de la plataforma.

- **Partner**: el cliente principal
- **Tenant / Affiliate**: un afiliado del cliente/partner
- **Clinic**:
	- MSO. Puede enviar prescripciones.
	- Es un grupo de doctores.
- **Client**:
	- ==Individuo que quiere vender pero no puede prescribir.==
	- Tiene acceso solo lectura al Admin Portal.
- **Questionnaire**: Health Assessment

## Portales 🌎

Al configurar Partners en esta plataforma se pueden activar varios portales según el tipo de cliente.

- **Patient Portal**: Eligibility y Screening + Checkout
- **Admin Portal**: Donde se pueden configurar Partners
- **Provider Portal**: donde pueden gestionar pacientes, screenings, consults, etc

### Tipos de Configuraciones

Son dos:

- Web App
- API o Headless

#### Configuración Web App

Es cuando el Partner no tiene equipo de tecnología ni website ni nada. En ese caso Launch le da todo.

- Portales de pacientes para que abran y completen el Eligibility y el Screening + Checkout.
- Admin Portal para gestionar todo con respecto a pacientes, screening, consults, etc.

> [!Warning]
> Cuando se configura un Partner tipo Web app se debe crear solo una App.
>
> Por defecto, todo Partner tiene acceso al API. En los casos de Partner con configuración Web App se tiene todo por defecto con una sola app.

Cuando se hace esta configuración se sigue la forma siguiente:

- 1 Partner
	- 1 Tenants / Affiliates
		- Para determinar el rol:
			- ==Si el Partner está necesitando una Web App, el rol es Client.==
			- ==Si el Partner está necesitando una API, el rol es Clinic.==
	- 1 Apps

> [!Warning]
> En una llamada, Jeff dijo que:
> > Cuando es Web App la propiedad `disable_provider_portal` debe ir en `true`, sin embargo, cuando se está probando en QA debe estar en `false`.
> 
> Esto lo veo en el caso de HappyGLP. El cambio que hice de lo de ocultar el botón de Refill lo hice en QA y estaba habilitado el Provider Portal.
> 
> Creo que debe deshabilitarse en prod es porque cualquier puede llegar y registrarse.



#### Configuración API o Headless 🚨

Es cuando el Partner sí tiene equipo de tecnología y solo le interesa integrarse con Launch. En este caso solo se le ofrece acceso web al Admin Portal.

> [!Warning]
> Cuando se configura un Partner tipo API/Headless se deben crear dos Apps.
>
> Esto es porque una ==App será de tipo API para dar acceso con secrets==. La otra App será para el acceso al Admin Portal.


Cuando se hace esta configuración se sigue la forma siguiente:

- 1 Partner
	- 2 Tenants / Affiliates
		- 1 tenant será Clinic y otro Client.
			- El Clinic es el que otorga los accesos
			- El Client es de solo lectura.
	- 2 Apps

#### Enlazar App a Tenant

TBC


## Catálogos

> [!Important]
> Se pueden reusar entre diferentes Apps.

El catálogo es en lo que se organizan los productos y sus variantes. Es lo que al final venden a los pacientes.

La taxonomía es:

- Catalog
	- Categories
		- Products
			- Variants

> [!Info]
> Cada Variant es lo que es un MedId en el MedPicker.
>
> MedPicker y HealthGrid/Launch trabajan por separado.

### Datos Sueltos

- Product attributes come from MedPicker
- ==Variant SKU es el mismo MedId==
- Price in a Variant is what is charged to the CX
- Price in Product is what is displayed in the Checkout


## Pruebas en QA/Local

Apuntes sueltos:
- Enlazar un practitioner es la única forma en que quede habilitado en Launch.
- En local/dev para poder completar Screening/Consult/Prescription hay que desactivar el `stub_mode` que aparece en el botón "Dev" en la esquina inferior derecha.