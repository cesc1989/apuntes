# Apuntes Ruby Parte 5

# Probando Enumerable#partition

La documentación dice que sirve para devolver un array de arrays, donde cada array contenga los elementos que cumplan con la condición del bloque.

Veamos ejemplos con lo que probé en Replit.

Para este conjunto de datos:

```ruby
personas = [
  { nombre: 'Francho', genero: 'masculino' },
  { nombre: 'Juana', genero: 'femenino' },
  { nombre: 'Pepita', genero: 'femenino' },
  { nombre: 'Felipe', genero: 'masculino' }
]
```

Si probamos así:

```ruby
r = personas.partition { |persona| persona[:genero] == 'masculino' }

[
    [0] [
        [0] {
            :nombre => "Francho",
            :genero => "masculino"
        },
        [1] {
            :nombre => "Felipe",
            :genero => "masculino"
        }
    ],
    [1] [
        [0] {
            :nombre => "Juana",
            :genero => "femenino"
        },
        [1] {
            :nombre => "Pepita",
            :genero => "femenino"
        }
    ]
]
```

También podemos devolver el resultado de la operación en dos variables así:

```ruby
hombres, mujeres = personas.partition { |persona| persona[:genero] == 'masculino' }

ap hombres

[
    [0] {
        :nombre => "Francho",
        :genero => "masculino"
    },
    [1] {
        :nombre => "Felipe",
        :genero => "masculino"
    }
]

ap mujeres

[
    [0] {
        :nombre => "Juana",
        :genero => "femenino"
    },
    [1] {
        :nombre => "Pepita",
        :genero => "femenino"
    }
]
```

Así que sí se ve bien útil como dice Collin en su [trino](https://x.com/collin_jilbert/status/1792545186825851184).

Enlace a los docs: https://ruby-doc.org/3.2.2/Enumerable.html#method-i-partition

Enlace al replit: https://replit.com/@cesc89/EnumerablePartition?v=1

# ¿Por qué es mejor anidar módulos en Ruby?

Esto es del artículo -> https://thoughtbot.com/blog/why-you-should-nest-modules-in-ruby

Normalmente hay dos opciones para escribir módulos en Ruby:

En línea:
```ruby
class SectionBuilder::V3::ExpansionPanel; end
```

Anidada/Nested
```ruby
module SectionBuilder
  module V3
    class ExpansionPanel
    end
  end
end
```

El artículo explica que es mejor anidar los módulos hará que Ruby sea más preciso al intentar buscar una clase anidada.

El caso del artículo me recuerda cuando en algunos proyectos me tocaba escribir algo como:
```ruby
::MyClass.new
```

para poder encontrar la clase fuera del anidamiento de módulos.

Da este ejemplo:
```ruby
module Admin
  class Nested
    def self.nesting = Module.nesting
  end
end

class Admin::Inline
  def self.nesting = Module.nesting
end

Admin::Nested.nesting #=> [Admin::Nested, Admin]
Admin::Inline.nesting #=> [Admin::Inline]
```

> ruby is aware that the Admin::Nested constant is defined within an Admin namespace. Whereas with the inline class, ruby considers the Admin::Inline class to just be on its own.

# ¿Cómo desinstalar versión de Ruby instalada con ruby-install?

Etiquetas: #ruby-install 

ruby-install hace dos cosas:

- descarga los archivos fuentes en la carpeta `~/src`
- instala las versiones de Ruby en la carpeta `~/.rubies/`

Todo lo que esté en `~/.rubies/` es leído por el comando `chruby` y lo mostrará como versión disponible para usar.

Así que para desinstalar una versión de Ruby instalada con ruby-install hay que eliminar la carpeta en ambos sitios:

- En `~/src` para limpiar el espacio en disco
- En `~/.rubies/` para desinstalar esa versión

Una vez eliminada de la carpeta `~/.rubies/` hay que recargar el shell para que no aparezca en las versiones listadas por `chruby`.

Visto en este [issue](https://github.com/postmodern/ruby-install/issues/135).

# Correr rubocop autocorrect para un cop en específico

Ejemplo para corregir Style/StringLiterals:
```
rubocop -a --only Style/StringLiterals
```

## Generar archivo .rubocop_todo.yml

```
rubocop --auto-gen-config
```