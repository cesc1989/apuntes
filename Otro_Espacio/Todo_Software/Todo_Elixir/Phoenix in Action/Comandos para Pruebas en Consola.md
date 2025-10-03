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

```erlang
Auction.User.changeset(
  %Auction.User{},
  %{username: "geo", email_address: "marikiti@gmail.com", password: "polvorete"}
)

Auction.User.changeset_with_password(
  %Auction.User{},
  %{username: "geo", email_address: "marikiti@gmail.com", password: "polvorete", password_confirmation: "polvorete"}
)
```