# Información Básica sobre Reviewers

Explicado en Notion [https://www.notion.so/getluna/Echo-07fe49e5a4c848a5be9d9669fa2b7573](https://www.notion.so/getluna/Echo-07fe49e5a4c848a5be9d9669fa2b7573)

Reviewers en Alpha: [https://echo.alpha.getluna.com/reviewers/sign_in](https://echo.alpha.getluna.com/reviewers/sign_in)

Reviewers en Local -> [http://localhost:3000/reviewers/sign_in](http://localhost:3000/reviewers/sign_in)

## Creando Reviewer en Local

```ruby
Reviewer.create(first_name: "Francisco", last_name: "Quintero", email: "francisco.quintero@ideaware.co")
```

## Aspecto de una Instancia en Local

```ruby
#<Reviewer:0x0000000116744de0> {
 "id" => 1,
"email" => "some@email.com",
"encrypted_password" => "$2a$11$w5ZQyN1//a50dfFs1WW99uvXYvJwsd6THnha8VdSONEd12x0PjVK6",
"reset_password_token" => nil,
"reset_password_sent_at" => nil,
"remember_created_at" => Tue, 25 Jul 2023 15:14:25.688317000 PDT -07:00,
"first_name" => "Zayter",
"last_name" => "Munive",
"created_at" => Wed, 30 Jan 2019 11:30:46.002359000 PST -08:00,
"updated_at" => Thu, 23 May 2024 06:03:36.654935000 PDT -07:00,
"hide_pii" => false,
"encrypted_otp_secret" => "7wCAVTQopHypEI8wcqAM0JsE7gHOjSvzPDVuPNiYux/vUOhbulZ86w==\n",
"encrypted_otp_secret_iv" => "pgPt7VCUs/YWgWcj\n",
"encrypted_otp_secret_salt" => "_LyCTKBu/YoQNEaaW1FfJuQ==\n",
"consumed_timestep" => 53380498,
"otp_required_for_login" => false,
"password_set_at" => Tue, 22 Mar 2022 09:23:18.144305000 PDT -07:00,
"password_set" => false,
"sign_in_count" => 155,
"current_sign_in_at" => Thu, 23 May 2024 06:03:36.654637000 PDT -07:00,
"last_sign_in_at" => Wed, 26 Jul 2023 13:03:06.336706000 PDT -07:00,
"current_sign_in_ip" => #<IPAddr: IPv4:190.61.61.202/255.255.255.255>,
"last_sign_in_ip" => #<IPAddr: IPv4:190.251.148.146/255.255.255.255>,
"provider" => "echo_google_oauth2",
"uid" => "104012756553115848641",
"unique_session_id" => "ALP8aMuj8sMr_KtGVqz3",
"encrypted_automation_token" => nil,
"automation_token_generated_at" => nil,
"automation_token_enabled" => false,
"otp_secret" => nil
```