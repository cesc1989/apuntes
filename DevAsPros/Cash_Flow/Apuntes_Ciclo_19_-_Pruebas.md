# Apuntes Ciclo 19 - Pruebas - Cash Flow

# Cómo probar que un elemento esté dentro de un contenedor

Estaba escribiendo pruebas para la páginas Explore y quería asegurarme que las pruebas fueran enfocadas al contenido de cada fila de gasto.

![[gasto.explore.png]]

Entonces aprovechando que cada gasto está contenido por un elemento `tr` puedo escribir la prueba de sistema así:
```ruby
within("tr#expenditure_#{january_exp.id}") do |h|
	expect(h).to have_text("Predial")
	expect(h).to have_text("$1,500,000")
end
```

Y así queda mejor representado el sentido de la prueba.