# Apuntes Ruby Parte 4

# Enumerable#each_with_object no mezcla hash vacío

Con un código como este:

```ruby
error_events.each_with_object({}) do |error, memo|
  memo.merge(
    dashboard_id: dashboard.id,
    user_id: dashboard.provider_id,
    provider_kind: dashboard.provider_kind,
    patient_id: error.extras['patient_id'],
    chart_id: error.extras['chart_id'],
    poc_id: error.extras['poc_id'],
    timestamp: error.date,
    error_location: error.extras['context']
  )
end
```

En cada iteración `memo` quedará vacío.

Para que `memo` sea un hash al final hay que ir asignando las llaves y valores con la otra sintaxis:

```ruby
error_events.each_with_object({}) do |error, memo|
  memo[:dashboard_id] = dashboard.id
  memo[:user_id] = dashboard.provider_id
  memo[:provider_kind] = dashboard.provider_kind
  memo[:patient_id] = error.extras['patient_id']
  memo[:chart_id] = error.extras['chart_id']
  memo[:poc_id] = error.extras['poc_id']
  memo[:timestamp] = error.date
  memo[:error_location] = error.extras['context']
  memo[:provider_email] = error.provider_email
end
```

**ATENCIÓN**
Parece que fue error mío. Hice esto en [replit](https://replit.com/@cesc89/eachwithobjecthashmerge#main.rb) y funcionó:

```ruby
r = ['a', 'b', 'c'].each_with_object({}).map do |letter, memo|
  memo.merge(letter => 'mundo')
end

pp r

=> [{"a"=>"mundo"}, {"b"=>"mundo"}, {"c"=>"mundo"}]
```

🤔 

# Obtener un porcentaje de 100% al sumar porcentajes de un mismo grupo

Al respecto:

- [How to convert percentage greater than 100% to 100%?](https://math.stackexchange.com/questions/3734065/how-to-convert-percentage-greater-than-100-to-100)
- [How to make rounded percentages add up to 100%](https://stackoverflow.com/questions/13483430/how-to-make-rounded-percentages-add-up-to-100)
- [How would you add percentages to always equal 100%](https://www.reddit.com/r/excel/comments/fh256g/how_would_you_add_percentages_to_always_equal_100/)

Pasó en Provider Portal al revisar la suma de porcentajes de los rangos de edades para el género femenino.

![No es el caso pero pongo la captura para referencia](https://paper-attachments.dropboxusercontent.com/s_C46A81088E2B80E0E571F8844AB37CEE513A9A7BC0D45F14A507383652F25D5C_1673534408676_image.png)


En un caso, teníamos este grupo de datos donde cada elemento del arreglo es un porcentaje. Al sumarlos, no dan 100%:

    Female: [1, 8, 28, 24, 16, 8, 10, 4, 2, 1, 0]
    # [1, 8, 28, 24, 16, 8, 10, 4, 2, 1, 0].reduce(&:+)
    # => 102 
    
    Male: [0, 3, 21, 22, 21, 7, 11, 7, 4, 3, 0]
    # [0, 3, 21, 22, 21, 7, 11, 7, 4, 3, 0].reduce(&:+)
    # => 99

Para resolverlo tuve que limitare el redondeo a dos caracteres:

```ruby
def percentage(value, total)
  return 0 if total.zero?

  (value * 100).fdiv(total).round(2)
end
```

Y se soluciona el problema en gran parte aunque en ocasiones las sumas pueden dar mayor a 100% al tener en cuenta los decimales.

Si solo uso la parte entera entonces no se llega a 100% ya que los decimales son los que hacen que se llegue con precisión a 100%.

Entonces nace la pregunta, ¿cómo obtener un 100% exacto de un grupo de porcentajes?

# Error instalar gema ffi en Macbook Chip M1

Cuando le hice bundle al proyecto Jekyll del sitio web de devaspros encontré este error de la gema FFI:

    Fetching ffi 1.10.0
    Installing ffi 1.10.0 with native extensions
    Gem::Ext::BuildError: ERROR: Failed to build gem native extension.
    
        current directory: /Users/francisco/.gem/ruby/2.7.7/gems/ffi-1.10.0/ext/ffi_c
    /Users/francisco/.rubies/ruby-2.7.7/bin/ruby -I /Users/francisco/.rubies/ruby-2.7.7/lib/ruby/2.7.0 -r ./siteconf20230226-72275-1n54fi5.rb
    extconf.rb
    
    An error occurred while installing ffi (1.10.0), and Bundler cannot continue.
    Make sure that `gem install ffi -v '1.10.0' --source 'https://rubygems.org/'` succeeds before bundling.
    
    In Gemfile:
      github-pages was resolved to 197, which depends on
        github-pages-health-check was resolved to 1.16.1, which depends on
          typhoeus was resolved to 1.3.1, which depends on
            ethon was resolved to 0.12.0, which depends on
              ffi
    

Una sugerencia era instalar la librería `libffi` con brew

    brew install libffi

pero ya estaba instalada.

Luego lo que [ayudó](https://www.moncefbelyamani.com/how-to-install-ffi-on-apple-silicon-m1-in-native-mode/) fue actualizar la gema

    bundle update ffi
    
    Using ffi 1.15.5 (was 1.10.0)

pero fallaba por culpa de nokogiri.


## Error instalar gema nokogiri Macbook M1

    Fetching nokogiri 1.10.1
    Installing nokogiri 1.10.1 with native extensions
    Gem::Ext::BuildError: ERROR: Failed to build gem native extension.
    
    An error occurred while installing nokogiri (1.10.1), and Bundler cannot continue.
    Make sure that `gem install nokogiri -v '1.10.1' --source 'https://rubygems.org/'` succeeds before bundling.
    
    In Gemfile:
      github-pages was resolved to 197, which depends on
        jekyll-mentions was resolved to 1.4.1, which depends on
          html-pipeline was resolved to 2.10.0, which depends on
            nokogiri

Según la [página de Nokogiri](https://nokogiri.org/tutorials/installing_nokogiri.html#installing-using-standard-system-libraries) se debería instalar así:

    brew install libxml2 libxslt
    gem install nokogiri --platform=ruby -- --use-system-libraries

Que funcionó pero notese el problema era que necesitaba instalar nokogiri 1.10.1 así que lo hice así:

    gem install nokogiri -v 1.10.1 --platform=ruby -- --use-system-libraries

y luego continué con el `bundle update ffi` y se completó el proceso.

# Cómo escribir un archivo pero agregando más líneas

De esta forma se escribe pero se borra el contenido anterior:

    open('myfile.txt', 'w') { |f| f << "some text or data structures..." } 

De esta forma se va a agregando el contenido en líneas nuevas:

    open('myfile.txt', "a") { |f| f << 'I am appended string' }

Visto en [Stack Overflow](https://stackoverflow.com/a/42169352/1407371).


# Enumerable#each_with_object solo devuelve un elemento de array a la vez

Todo esto tal vez fue confusión mía, sin embargo, aquí lo expongo.

Tenía este código en Provider Portal para exportar unos datos

```ruby
data = Dashboard.fully_loaded.map do |dashboard|
  dashboard.emails.each_with_object({}) do |email, memo|
    # vars
    memo[:emails] = email
    memo[:dashboard_id] = dashboard.id
    
    # more attributes
  end
end
```

El problema con este código era que para un conjunto de datos con esta forma

    ["steve@getluna.com"],
    ["fabricio.puche@ideaware.co"],
    ["foo@bar.com", "ivan.barona@koombea.com"],
    ["francisco.quintero+11@ideaware.co",
     "francisco.quintero+33@ideaware.co",
     "francisco.quintero+44@ideaware.co"]

Si quería ejecutar una acción por cada email en cada array, el proceso se ejecutaría pero el resultado final según el bloque de `each_with_object` sería solo un elemento. A pesar de que el array tuviera más.

Veamos.

Con este código

```ruby
r = Dashboard.fully_loaded.map do |dashboard|
  r1 = dashboard.emails.each_with_object({}) do |email, memo|
    memo[:emails] = email
  end
  pp r1
end
```

Obtendría

    {:emails=>"steve@getluna.com"}
    {:emails=>"fabricio.puche@ideaware.co"}
    {:emails=>"ivan.barona@koombea.com"}
    {:emails=>"francisco.quintero+44@ideaware.co"}


# Creando subclasses con Struct

Struct es más veloz que OpenStruct aunque menos flexible. Ya sé cómo hacer mocks con OpenStruct y también quería aprender con Struct.

Si quisiera replicar una estructura como esta

```ruby
Class Aws::Athena::Types::GetQueryResultsOutput
  result_set = Class Aws::Athena::Types::ResultSet
    rows = [
      [0] Class Aws::Athena::Types::Row
        data = [
          [0] Class Aws::Athena::Types::Datum
            var_char_value = 'physician_name',
          [1] Class Aws::Athena::Types::Datum
            var_char_value = 'physician_group'
        ]
      [1] Class Aws::Athena::Types::Row
        data = [
          [0] Class Aws::Athena::Types::Datum
            var_char_value = '',
          [1] Class Aws::Athena::Types::Datum
            var_char_value = ''
        ]
    ]
```

Donde el objetivo es poder replicar hasta la clase ResultSet, podría hacerlo así con Struct

    result_set_mock = Struct.new(:nothing) do
      def rows
        []
      end
    end
    result_set_mock.new(1)

Idea por haber visto [este artículo.](https://www.leighhalliday.com/ruby-struct)


# Instalando eventmachine 1.2.7 en Mac M1

Necesitaba actualizar Jekyll para el proyecto de mi sitio web y tenía este error de Event Machine:
```bash
 An error occurred while installing eventmachine (1.2.7), and Bundler cannot continue.
	
	In Gemfile:
		minima was resolved to 2.5.1, which depends on
			jekyll-feed was resolved to 0.12.1, which depends on
				jekyll was resolved to 4.1.1, which depends on
					em-websocket was resolved to 0.5.3, which depends on
						eventmachine
```

Probé, según mencionan en [este issue](https://github.com/eventmachine/eventmachine/issues/932#issuecomment-864392814) este comando
```bash
gem install eventmachine -- --with-openssl-dir=/usr/local/opt/openssl@1.1

4 warnings generated.
linking shared-object rubyeventmachine.bundle
ld: warning: directory not found for option '-L/usr/local/opt/openssl@1.1/lib'
Undefined symbols for architecture arm64:
	"_SSL_get1_peer_certificate", referenced from:
			SslBox_t::GetPeerCert() in ssl.o
ld: symbol(s) not found for architecture arm64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [rubyeventmachine.bundle] Error 1

make failed, exit code 2
```

Que no sirvió. Ni este otro:
```bash
gem install eventmachine -- --with-openssl-dir=/opt/homebrew/opt/openssl@1.1

4 warnings generated.
linking shared-object rubyeventmachine.bundle
Undefined symbols for architecture arm64:
	"_SSL_get1_peer_certificate", referenced from:
			SslBox_t::GetPeerCert() in ssl.o
ld: symbol(s) not found for architecture arm64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [rubyeventmachine.bundle] Error 1

make failed, exit code 2
```

Finalmente, con lo que comparten [este artículo](https://www.jessesquires.com/blog/2023/01/18/eventmachine-failure-on-macos-ventura/), fue que pude instalarlo la gema:
```bash
gem install eventmachine -v '1.2.7' -- --with-ldflags="-Wl,-undefined,dynamic_lookup"
	
	Building native extensions with: '--with-ldflags=-Wl,-undefined,dynamic_lookup'
	This could take a while...
	Successfully installed eventmachine-1.2.7
	Parsing documentation for eventmachine-1.2.7
	Installing ri documentation for eventmachine-1.2.7
	Done installing documentation for eventmachine after 2 seconds
	1 gem installed
```


# Diferencia entre tap y yield_self

Fuente: [monoRails](https://monorails.substack.com/p/ruby-tap-vs-yield_self).

## tap

> tap() method yields self to the block and returns self

Pasa la instancia al bloque y se retorna así misma.

**¿Cómo se usa normalmente?**

> One of the typical use cases of tap() is the method chaining over an object, operating over it inside a block, and returning the object itself.

**Ejemplos**

```ruby
new_product = Product.new.tap do |product|
  product.name = "iphone11"
  product.description = "iphone11 description"
end

new_product.name
=> "iphone11"
```

En este caso, en el bloque se modifica el objeto y se retorna modificado.

> Recuerda: `tap` siempre devolverá el objeto sobre el cual se realiza la operación. Si se necesita el resultado de la operación, `tap` no sirve para esto.

## yield_self

> yield_self() method passes self to the block and returns the result of it.

Similar a `tap` pero en lugar de devolver la instancia, devuelve el resultado.

**Ejemplos**

```ruby
new_product = Product.new.yield_self do |product|
  product.name = "iphone11"
  product.description = "iphone11 description"
end

=> "iphone11 description"
```

## En resumen

Ambos métodos sirven para hacer una operación en el objeto y, según el caso, devolver:

- `tap` devuelve la instancia
- `yield_self` devuelve el resultado de la operación

