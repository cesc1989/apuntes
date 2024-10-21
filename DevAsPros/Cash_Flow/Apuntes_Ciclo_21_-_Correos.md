# Apuntes Ciclo 21 - Envío de Correos

# Correos no salen desde el servidor

Pasaba este error:
```
Delivered mail 671664e31ebd0_29dda4ab0200cd@localhost.mail (2.9ms)
/home/ubuntu/cashflow/deployments/api-gems/bundle/ruby/3.1.0/gems/net-smtp-0.5.0/lib/net/smtp/authenticator.rb:20:in `check_args': SMTP-AUTH requested but missing user name (ArgumentError)
```

Y era porque tenía mal configurada la variable de entorno para el user name.

Antes:

```
export tMAILGUN_USERNAME=
```

Después:

```
export MAILGUN_USERNAME=
```

Ahora el error es otro:

```
Delivered mail 6716653cba6a7_29eca4ab06316b@localhost.mail (168.6ms)

/home/ubuntu/cashflow/deployments/api-gems/bundle/ruby/3.1.0/gems/net-smtp-0.5.0/lib/net/smtp.rb:1042:in `check_continue': could not get 3xx (421: 421 Domain sandbox947896f1956b403197c80061aa04c9e4.mailgun.org is not allowed to send: Sandbox subdomains are for test purposes only. Please add your own domain or add the address to authorized recipients in Account Settings. (Net::SMTPUnknownError)

)
```

# Configuración subdominio en Namecheap para envío de correo desde Mailgun

La clave de la configuración es que cuando se creen los registros TXT no hay que poner el nombre del dominio en el campo Host.

Esto no:
```
pic._domainkey.mg.devaspros.com
```

Eso sí:
```
pic._domainkey.mg
```

Otro ejemplo.

Así no:
```
mg.devaspros.com
```

Así sí:
```
mg
```

Para el registro CNAME la dinámica es similar para el campo Host.

Esto no:
```
email.mg.devaspros.com
```

Eso sí:
```
email
```