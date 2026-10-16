# Bundle y Gemas

Etiquetas: #gemfile 

Nota para encontrar cosas clave de uso de Bundle y de las gemas que uso en Ruby y/o Rails.

## Actualizar gema que no está listada en el Gemfile

Ver: [[Apuntes_Ruby_Parte_5#Actualizar gema no listada en Gemfile 🔑]]

Resumen: se logra usando la bandera `--conservative`.

Enlaces:

- [Makandra](https://makandracards.com/makandra/13885-update-single-gem-conservatively)
- [Docs de Bundle](https://bundler.io/man/bundle-update.1.html)

Así actualicé la gema Addressable de la versión 2.8.7 a la 2.9.0:
```bash
bundle update addressable --conservative
```

El diff:
```diff
-- addressable (2.8.7)
--	 public_suffix (>= 2.0.2, < 7.0)
++ addressable (2.9.0)
++	 public_suffix (>= 2.0.2, < 8.0)
```

## Ser más preciso con la versión a Instalar

> [!Important]
> Se logra con las banderas `--major`, `--minor` o `--patch`.
>
> Ver: https://bundler.io/man/bundle-update.1.html#PATCH-LEVEL-OPTIONS

En el caso anterior de addressable se actualizó por defecto a la siguiente versión. En este caso fue la minor.

Así funca Semantic Versioning:
> Given a version number MAJOR.MINOR.PATCH, increment the:
>
>  1. MAJOR version when you make incompatible API changes
>  2. MINOR version when you add functionality in a backward compatible manner
>  3. PATCH version when you make backward compatible bug fixes

Sitio web: https://semver.org/

## Warning de constantes al correr comandos de bundle

Esto me salía en varios proyectos luego de haber actualizado Ruby Gems:
```bash
warning: previous definition of X64_MINGW was here
warning: already initialized constant Gem::Platform::WINDOWS
warning: already initialized constant Gem::Platform::X64_LINUX_MUSL
```

El problema es la versión de bundle que está en el Gemfile:
```ruby
BUNDLED WITH
   2.4.22
```

la cual es muy vieja para la versión de Bundle a la que pasé (4.0.21). Para corregir hay que correr:
```bash
bundle update --bundler
```

Lo que cambiará la versión de bundler en el Gemfile:
```diff
BUNDLED WITH
-   2.4.22
+   4.1.0.beta2
```

Además, cambié platfform de la gema byebug para que use las correctas:
```ruby
gem "byebug", platforms: %i[mri windows]
```

porque había un warning de depreciación sobre eso.