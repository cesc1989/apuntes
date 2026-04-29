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
> Se logra con las banderas `--major`, `--minor` o `--patch`

