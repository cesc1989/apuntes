# Configurando Backend/Edge en Mac M1

# Instalando gema glib2 version 4.0.6

Todo va bien hasta que el bundle install falla por la gema glib2.
```
An error occurred while installing glib2 (4.0.6), and Bundler cannot continue.

In Gemfile:
  poppler was resolved to 4.0.6, which depends on
	cairo-gobject was resolved to 4.0.6, which depends on
	  glib2
```

Intentó instalar la gema manualmente pero nada funciona.
```
gem install glib2 -v 4.0.6 -- --disable-march-tune-native
Building native extensions with: '--disable-march-tune-native'
This could take a while...
ERROR:  Error installing glib2:
ERROR: Failed to build gem native extension.

	current directory: /Users/francisco/.gem/ruby/3.0.4/gems/glib2-4.0.6/ext/glib2
```

O así:
```
gem install glib2 -v 4.0.6
Building native extensions. This could take a while...
ERROR:  Error installing glib2:
ERROR: Failed to build gem native extension.

	current directory: /Users/francisco/.gem/ruby/3.0.4/gems/glib2-4.0.6/ext/glib2
```

Ya con Brew están instaladas dependencias:
```
brew install pkg-config
Warning: pkg-config 0.29.2_3 is already installed and up-to-date.
To reinstall 0.29.2_3, run:
  brew reinstall pkg-config

brew install glib
Warning: glib 2.76.3 is already installed and up-to-date.
To reinstall 2.76.3, run:
  brew reinstall glib
```


## Solución

Tocó cambiar la versión de Poppler a la 4.1.7 → https://rubygems.org/gems/poppler/versions/4.1.7

# Otras configuraciones

Copiar el archivo .env.example y luego agregar variables de entorno faltantes:

    cp .env.example .env

Las variables faltantes:
```bash
NEW_WORLD_HTTP_AUTH_USERNAME="hola"
NEW_WORLD_HTTP_AUTH_PASSWORD="hola"

GOOGLE_OAUTH_ID="ASK_RYAN"
GOOGLE_OAUTH_SECRET="ASK_RYAN"
```

# Crear AdminUser

Tengo que crear un AdminUser con mi correo para el login con Google
```ruby
AdminUser.create(
  first_name: 'Francisco',
  last_name: 'Quintero',
  region_id: Region.first.id,
  email: 'francisco.quintero@ideaware.co',
  is_active: true
)
```

## Permisos que podría necesitar

- `patient_creation_flow_enabled`

```ruby
AdminUser.find_by(email:"francisco.quintero@ideaware.co").update(super_admin:true)

AdminUser.find_by(email:"francisco.quintero@ideaware.co").update(therapist_credentials_management_enabled:true)
```

# Verificación manual de correo de portal

```ruby
Clinic.find('fd934542-80dd-48a4-8e1e-eb1258a6f028').portal_email_recipients.first.email_address.update(verified_at: Date.today, verification_status: 'verified')
```

Los modelos pertinentes son:

- UserCommunicationMethod
- ShadowUser
- PortalProviderEntity


# Archivos env locales

El archivo .env.test depende de que el usuario de la base de datos sea postgres. En mi caso no es así y cada que quiero correr las pruebas me toca cambiarlo.

Para esto puedo crear un archivo `.env.test.local` el cual va a reemplazar, en los valores que ahí defina, a los que están en `.env.test`. De esa forma me evito ese error.

Este sería el archivo:

```
# .env.test.local
DATABASE_USERNAME=francisco
HUBSPOT_ACCESS_TOKEN=
```

# Notas sobre trabajar en Luxe

- Si mi rama de trabajo lleva mucho tiempo viva, lo mejor es hacer una nueva más reciente y mover mis commits a esa nueva
- Revisa los valores que deben haber en `.env` vs `.env.example`. A veces van a faltarme envs.
- Cuando falten migraciones en TEST, lo mejor va a ser borrar toda la base de datos completa:
    - comando `rails db:setup`
    - Ahora que se usa la bd generada por RDS, es más fácil hacer esto

