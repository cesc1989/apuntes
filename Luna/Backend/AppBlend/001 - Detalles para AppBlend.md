# Detalles

Edge actualmente está en **Rails 6.1.6**

- Objetivo: **7.0.4**
- Versiones que siguen de la 6.x.x
	- 6.1.6.1 (Jul 12, 2022) - [Release page](https://github.com/rails/rails/releases/tag/v6.1.6.1)
	- 6.1.7 (Sep 9, 2022) - [Release page](https://github.com/rails/rails/releases/tag/v6.1.7)
	- 6.1.7.1 (Jan 17, 2023) - [Release page](https://github.com/rails/rails/releases/tag/v6.1.7.1)
	- 6.1.7.2 (Jan 24, 2023) - [Release page](https://github.com/rails/rails/releases/tag/v6.1.7.2)
	- 6.1.7.3 (Mar 13, 2023) - [Release page](https://github.com/rails/rails/releases/tag/v6.1.7.3)
	- 6.1.7.4 (Jun 26, 2023) - [Release page](https://github.com/rails/rails/releases/tag/v6.1.7.4)
	- 6.1.7.5 (Aug 22, 2023) - [Release page](https://github.com/rails/rails/releases/tag/v6.1.7.5)
	- 6.1.7.6 (Aug 22, 2023) - [Release page](https://github.com/rails/rails/releases/tag/v6.1.7.6)
	- 6.1.7.7 (Feb 21, 2024) - [Release page](https://github.com/rails/rails/releases/tag/v6.1.7.7)
	- 6.1.7.8 (Junio 4, 2024) - [Release page](https://github.com/rails/rails/releases/tag/v6.1.7.8)
- Versiones que siguen de la 7.0.x
	- 7.0.0 (Dec 15, 2021) - [Release page](https://github.com/rails/rails/releases/tag/v7.0.0)
	- 7.0.1 (Jan 6, 2022) - [Release page](https://github.com/rails/rails/releases/tag/v7.0.1)
	- 7.0.2 (Feb 8, 2022) - [Release page](https://github.com/rails/rails/releases/tag/v7.0.2)
	- 7.0.3 (May 9, 2022) - [Release page](https://github.com/rails/rails/releases/tag/v7.0.3)
	- 7.0.4 (Sep 9, 2022) - [Release page](https://github.com/rails/rails/releases/tag/v7.0.4)

[Página de versiones de Rails en GitHub](https://github.com/rails/rails/tags)

## Cambios Clave en Versión 7.0.0

Ver [Release page](https://github.com/rails/rails/releases/tag/v7.0.0)

### Active Support

Deprecate passing a format to `#to_s` in favor of `#to_formatted_s` in `Array`, `Range`, `Date`, `DateTime`, `Time`,  
`BigDecimal`, `Float` and, `Integer`.

### Railtie

The setter `config.autoloader=` has been deleted. `zeitwerk` is the only  
available autoloading mode.

During initialization, you cannot autoload reloadable classes or modules  
like application models, unless they are wrapped in a `to_prepare` block.  
For example, from `config/initializers/*`, or in application, engines, or  
railties initializers.

> Please check the [autoloading  guide](https://guides.rubyonrails.org/v7.0/autoloading_and_reloading_constants.html#autoloading-when-the-application-boots) for details.

Fix compatibility with `psych >= 4`.

> Starting in Psych 4.0.0 `YAML.load` behaves like `YAML.safe_load`. To preserve compatibility  `Rails.application.config_for` now uses `YAML.unsafe_load` if available.

## Completo

[Página de versiones de Ruby](https://www.ruby-lang.org/en/downloads/releases/)

El 17 de Julio de 2024 Edge fue actualizado a Ruby 3.1.0.

En Ruby: **Ruby 3.0.6**

- Objetivo: **3.1.0**
- Las versiones que le siguen: 
	- 3.0.7
	- 3.1.0