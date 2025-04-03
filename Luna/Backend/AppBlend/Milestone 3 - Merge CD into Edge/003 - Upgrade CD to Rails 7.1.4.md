# Upgrade Clinical Dashboard to Rails 7.1.4

# Error con gema debug

Daba este error:
```bash
Failure/Error:
       def reset(prompt = '', encoding:)
         super
         Signal.trap(:SIGWINCH, nil)

     ArgumentError:
       missing keyword: :encoding
     # /Users/francisco/.gem/ruby/3.1.6/gems/debug-1.8.0/lib/debug/console.rb:25:in `reset'
```

Al usar `debugger` en algún archivo de pruebas. Tocó hacer upgrade a la [versión 1.10.0](https://github.com/ruby/debug/releases/tag/v1.10.0).

# Invalid HTTP status: :unprocessable_entity

¿Qué? Llevo un montón de años usando ese estado con [ese símbolo](https://gist.github.com/mlanett/a31c340b132ddefa9cca) y ahora resulta que no es válido? Pues sí.

En Rack 3 el nombre cambió a "Unprocessable Content". Puedo ver que la respuesta de la petición en la prueba sigue devolviendo 422 pero ahora Rack usa otro nombre:
```ruby
Rack::Utils::HTTP_STATUS_CODES
{
 # ...
 421=>"Misdirected Request",
 422=>"Unprocessable Content",
 423=>"Locked",
 424=>"Failed Dependency",
 # ...
}
```

Una vez lo cambio a `unprocessable_content` las pruebas pasan.

Explicado en [rspec-rails](https://github.com/rspec/rspec-rails/issues/2763).