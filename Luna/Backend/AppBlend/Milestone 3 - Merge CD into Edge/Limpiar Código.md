# Código que se puede borrar una vez haga el merge

Lo puedo hacer antes, durante o después. Idealmente, durante.

## En BaseApiController

Quitar la rama `elsif current_user.present?` de `infer_token`. Eso ya no se usa.

## En routes/clinical_dashboard.rb

Quitar la ruta para cargar /dashboard. Eso era necesario cuando se usaba Webpacker.

```ruby
# The constraint is because rails doesn't allow dots
# because that's the format separator
#
# See https://guides.rubyonrails.org/routing.html#specifying-constraints
resources :dashboard,
	only: [:index, :show],
	param: :token,
	constraints: { token: /[^\/]+/ }
```

