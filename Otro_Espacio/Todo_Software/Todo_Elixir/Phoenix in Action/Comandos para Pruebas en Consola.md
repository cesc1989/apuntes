# Comandos para Copiar y Pegar para Probar

## Probar Auction.Item

Para probar el módulo Item.

```erlang
Auction.insert_item(
  %{
    title: "Hola mundo 2",
    description: "holllaaa mundito",
    ends_at:  DateTime.from_naive!(~N[1980-01-01 00:00:00],"Etc/UTC")
  }
)

item = Auction.get_item(3)

Auction.Item.changeset(
  %Auction.Item{},
  %{
    title: "Prueba extensa",
    description: "Soy una buena descripción descrita.",
    ends_at: ~N[2024-12-25 00:00:00]
  }
)

item
|> Auction.Item.changeset(
  %{
    title: "Prueba extensa",
    description: "Soy una buena descripción descrita.",
    ends_at: ~N[2024-12-25 00:00:00]
  }
)
|> Auction.Repo.update()
```

## Probar Auction.User

Para probar el módulo User.

`changeset/2` no tiene en cuenta `password`:
```erlang
Auction.User.changeset(
  %Auction.User{},
  %{username: "geo", email_address: "marikiti@gmail.com", password: "polvorete"}
)
```

Por lo que devuelve:
```erlang
#Ecto.Changeset<
  action: nil,
  changes: %{username: "geo", email_address: "marikiti@gmail.com"},
  errors: [hashed_password: {"can't be blank", [validation: :required]}],
  data: #Auction.User<>,
  valid?: false,
  ...
>
```

Mientras que `changeset_with_password` sí lo hace:
```erlang
Auction.User.changeset_with_password(
  %Auction.User{},
  %{username: "geo", email_address: "marikiti@gmail.com", password: "polvorete", password_confirmation: "polvorete"}
)
```

Así que la cosa cambia y ahora sí se podría guardar en la BD.
```erlang
#Ecto.Changeset<
  action: nil,
  changes: %{
    username: "geo",
    password: "polvorete",
    email_address: "marikiti@gmail.com",
    hashed_password: "$pbkdf2-sha512$160000$R3yCJ.XX9l5LUYLwARGk3g$eoL06Aipybzd8Fwhw3XSP7WeM6edEF1Ke1SfU5hxddL6CAEXv5A.S2cvtEuXI4DhLMmdbjllfbJ8EtjDK5l9XA"
  },
  errors: [],
  data: #Auction.User<>,
  valid?: true,
  ...
>
```