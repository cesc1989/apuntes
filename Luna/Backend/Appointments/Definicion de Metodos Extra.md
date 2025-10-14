# Definición de Métodos Extra

Las instancias de Appointment tienen métodos como:

- `initial_visit?`
- `progress_visit?`

Sin embargo, en ninguna parte del modelo están definidos. Al buscar encontré esto en la mezcla de métodos de instancia:
```ruby
VISIT_TYPES.each do |type_of_visit|
	define_method(:"#{type_of_visit}_visit?") do
		self.visit_type == type_of_visit
	end
end
```

Esta esta metaprogramación la que le da vida a esas funciones.

> [!Note]
> Ver los visit types en [[Crear Appointments en Alpha]]