# Manejo de Moneda en Base de datos es con la sub-unidad: centavos o centimos

En vez de usar la denominación en decimales de la plata que se cobra o ingresa mediante aplicaciones web, es una práctica establecida y bien aceptada el usar enteros.

Es decir, en lugar de almacenar 10 dólares se almacenan 1000 centavos.

Las razones para esto son técnicas y prácticas. A continuación describo varias de estas.

## Precisión en cálculos Financieros

Los números decimales pueden dar problemas la hacer cálculos matemáticos. Estos puede producir errores de redondeo. Se puede perder o cobrar dinero adicional.

Para minimizar la posibilidad de error, por ejemplo, se almacenan 10.50 como 1050 centavos.

Este valor iría en un campo de tipo `integer` o `bigint`.

## Simplificación de operaciones

A la práctica, es mucho más sencillo hacer operaciones entre enteros. Hay más consistencia en trabajar con enteros que con punto flotantes (donde toca decidir cuántos dígitos después de la coma usar).

## Evitar problemas de redondeo

Es más sencillo redondear enteros.

Este ejemplo lo generó DeepSeek:
```ruby
# Precios de los productos en dólares
prices_in_dollars = [10.99, 5.50, 7.25]

# Método 1: Usando decimales (dólares)
total_in_dollars = prices_in_dollars.sum
tax_in_dollars = total_in_dollars * 0.0825
total_with_tax_dollars = total_in_dollars + tax_in_dollars

# Método 2: Usando centavos (enteros)
prices_in_cents = prices_in_dollars.map { |price| (price * 100).round }
total_in_cents = prices_in_cents.sum
tax_in_cents = (total_in_cents * 0.0825).round
total_with_tax_cents = total_in_cents + tax_in_cents

# Convertir el total en centavos de vuelta a dólares
total_with_tax_cents_in_dollars = total_with_tax_cents / 100.0

# Resultados
puts "Método 1 (decimales):"
puts "Total sin impuesto: $#{total_in_dollars.round(2)}"
puts "Impuesto (8.25%): $#{tax_in_dollars.round(2)}"
puts "Total con impuesto: $#{total_with_tax_dollars.round(2)}"
puts ""

puts "Método 2 (centavos):"
puts "Total sin impuesto: $#{(total_in_cents / 100.0).round(2)}"
puts "Impuesto (8.25%): $#{(tax_in_cents / 100.0).round(2)}"
puts "Total con impuesto: $#{total_with_tax_cents_in_dollars.round(2)}"
```

Explica que al final ambos casos arrojan el mismo resultado:
```
# Decimales
Total sin impuesto: $23.74
Impuesto (8.25%): $1.96
Total con impuesto: $25.70

# Enteros
Total sin impuesto: $23.74
Impuesto (8.25%): $1.96
Total con impuesto: $25.70
```

Pero aclara que:

- Los problemas de redondeo suelen aparecer en cálculos más complejos o con más decimales
- Si el impuesto fuera `8.2525%`, el método 1 podría dar un resultado como `$1.960123` debido a la representación de punto flotante, mientras que el método 2 redondearía directamente a `196` centavos (`$1.96`)
- En operaciones repetitivas (como sumar muchos productos), los errores de redondeo pueden acumularse al trabajar con decimales, pero no con enteros


## Compatibilidad con sistemas legacy

Sistemas antiguos ya usan la sub-unidad de la moneda para guardar los valores.

# En el uso

Para guardar un valor en decimal a su representación en entero se multiplica por 100. Cuando se quiere mostrar al usuario en una UI se divide el valor en entero entre 100.

Ejemplo.

Una compra de 25.99 USD.

Para guardar en la base de datos o enviar a un sistema de pagos se multiplica el valor por 100:
```ruby
25.99 * 100 = 2599
```

Cuando queremos mostrar dicho valor en centavos al usuario, se divide entre 100:
```ruby
2599 / 100 = 25.99
```