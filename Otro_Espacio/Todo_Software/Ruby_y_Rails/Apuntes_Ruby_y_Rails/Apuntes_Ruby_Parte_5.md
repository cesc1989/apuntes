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

