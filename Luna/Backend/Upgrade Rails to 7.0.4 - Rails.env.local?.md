# Upgrade Rails to 7.0.4 - Rails.env.local?

## Particularidades

Edge actualmente está en:

**Rails 6.1.6**
- Objetivo: 7.0.4
- Las versiones que le siguen:
	- 6.1.6.1
	- 6.1.7
	- 6.1.7.1
	- 6.1.7.2
	- 6.1.7.3

**Ruby 3.0.6**
- Objetivo: 3.1.0
- Las versiones que le siguen: 
	- 3.0.7
	- 3.1.0

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

# Compatibilidad entre Rails y Ruby

Encontré [esta tabla](https://www.fastruby.io/blog/ruby/rails/versions/compatibility-table.html), actualizada a fecha de **24 de Abril de 2024**, sobre la compatibilidad entre versiones de Rails y de Ruby.

![[Pasted image 20240704234741.png]]

Conclusiones:
- Puedo subir a Rails 7.0.4 y a Ruby 3.1.0

Notas:

Aquí [mencionan](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html#ruby-versions) primero subir la versión de Ruby y luego la de Rails

> It's a good idea to upgrade Ruby and Rails separately. Upgrade to the latest Ruby you can first, and then upgrade Rails.