# Rails.env.local? en Rails 7.1

Edge tiene su propia definición que está en `lib/lunacare/environment.rb`:

```ruby
elsif Rails.env.development?
    EnvironmentInquirer.new("local")
```

Sin embargo, en Rails 7.1 se [introdujo esta misma](https://blog.saeloun.com/2023/06/26/rails-env-local/) característica.

Mira el [PR donde](https://github.com/rails/rails/pull/46786) se agrega este cambio.

```
Add `Rails.env.local?` shorthand for `Rails.env.development? || Rails.env.test?
```

Para la actualización a Rails 7.0.4 no debería afectar tener esta clase personalizada porque no habría conflicto.