# Lecciones Ruby - Luna Patients

# Sobre memoization de variables de instancia en métodos que se llaman varias veces

Este caso se dio en la clase `PatientForm`. Cuando intenté cambiar esto:

```ruby
def patient
  @patient ||= Patient.new(@params)
end
```

Por:

```ruby
def patient
  @patient = Patient.find_or_initialize_by(internal_id: @params[:internal_id])
  @patient.attributes = @params
  @patient
end
```

Los tests no pasaban y la respuesta JSON no era la que contenía los errores.

El problema ocurre porque el método `patient` se estaba llamando en varios lugares. En mi versión sin memoización `||=` cada llamado a `patient` crea una nueva instancia. Esto lo pude comprobar al [imprimir el atributo](https://ruby-doc.org/core-2.5.3/Object.html#method-i-object_id) `[object_id](https://ruby-doc.org/core-2.5.3/Object.html#method-i-object_id)` [de cada instancia](https://ruby-doc.org/core-2.5.3/Object.html#method-i-object_id).

```
    Started POST "/patient_forms/" for 127.0.0.1 at 2019-06-27 16:25:04 -0500
    Processing by Api::V1::PatientFormsController#create as JSON
      Parameters: {"patient"=>{"first_name"=>nil, "last_name"=>nil, "internal_id"=>"99993", "forms_attributes"=>[{"type_name"=>nil, "injury_name"=>nil}]}, "patient_form"=>{"patient"=>{"first_name"=>nil, "last_name"=>nil, "internal_id"=>"99993", "forms_attributes"=>[{"type_name"=>nil, "injury_name"=>nil}]}}}
    Can't verify CSRF token authenticity.
    70098065484300
      Patient Exists (0.4ms)  SELECT  1 AS one FROM "patients" WHERE "patients"."internal_id" = $1 LIMIT $2  [["internal_id", "99993"], ["LIMIT", 1]]
    70098028430460
       (0.2ms)  BEGIN
      CACHE Patient Exists (0.0ms)  SELECT  1 AS one FROM "patients" WHERE "patients"."internal_id" = $1 LIMIT $2  [["internal_id", "99993"], ["LIMIT", 1]]
       (0.2ms)  ROLLBACK
    [active_model_serializers] Rendered ActiveModel::Serializer::Null with Hash (0.06ms)
    Completed 422 Unprocessable Entity in 54ms (Views: 6.6ms | ActiveRecord: 0.8ms)
```

Una instancia tenía el *object_id* **70098065484300** y otra tenía el ID **70098028430460.**

La solución fue memoizar la asignación cómo se estaba haciendo originalmente.

> En realidad la solución sería tener la variable `@patient` al instanciar el `PatientForm` pero en este caso pasamos por alto.

```ruby
def patient
  @patient ||= Patient.find_or_initialize_by(internal_id: @params[:internal_id])
  @patient.attributes = @params
  @patient
end
```

# Extraer valores de un hash usando un array de llaves

Teniendo un hash con X cantidad de llaves, cuando quiero traer solo ciertas llaves pero de una forma dinámica, ¿cómo se haría?

Si tengo por ejemplo el hash:

```ruby
hsh = { a: 1, b: 2, c: 3, d: 4, e: 5}
```

Y solo quisiera traer unos cuantas en un nuevo hash pero sin tener que escribirlo de nuevo, podría hacer algo como lo que [se describe en Coderwall](https://coderwall.com/p/sshdkg/extract-key-value-pairs-from-a-hash-in-ruby-using-an-array-of-keys).

```ruby
keys = ['a', 'b']
    
new_hsh = hsh.select { |key| hsh.include?(key) }
```

En el ejemplo en Coderwall se usa `#reject` en vez de `#select` pero con este último se lee un poco más simple la ejecución.

En Luna Patients Forms, usé está técnica en la clase `Forms::IntakeFormBuilder`

# Sobre File.read

Resulta que en Edge estoy haciendo esto:

```ruby
file = download_signed_terms(patient_id) do |tempfile|
  File.read(tempfile)
end
```

La línea `File.read` la copié de otro código en el mismo proyecto.

He tenido varias dificultados y me puse a buscar la documentación de ese método de clase y [no está documentado en la clase File](https://ruby-doc.org/core-3.1.0/File.html). Existe pero en [la clase IO](https://ruby-doc.org/core-3.0.2/IO.html#method-c-read).

Y eso no me sirve para el código anterior porque está abriendo el archivo y sacando los contenidos. Lo que necesito es el archivo solo.

# Cómo verificar si un String tiene un Integer?

Resulta que para la versión 3 del API podía darse estas variaciones:
```ruby
component_id = 45454
component_id = "45454"
component_id = ""
component_id = nil
component_id = "some-uuuu-iii-dddd"
```

pero la única variante incorrecta sería la que tuviera un UUID o un string que no contuviera un entero. Entonces necesitaba una forma de verificar que se diera la condición y encontré que se logra con una expresión regular:

```ruby
"1234223".match? /\A\d+\z/
#=> true
```

Y lo terminé usando así:
```ruby
component_id.match?(/\A\d+\z/)
```

Visto en [Stack Overflow](https://stackoverflow.com/a/18846004/1407371).