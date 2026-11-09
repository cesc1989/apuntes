# ADR-002 Usar NS de DNSimple

Namecheap ofrece la compra de dominios a un precio menor que varios proveedores. También ofrece muchos productos más. Uno de esos productos es Name Servers (NS) de manera gratuita.

Desde el admin de dominios se pueden configurar muchos aspectos incluidos la configuración de DNS. Si bien se pueden configurar todo sin problemas, la UI de Namecheap es muy cargada y llena de upsells. El negocio de Namecheap es dominios pero ofrecen muchas cosas más. Todo eso crea un lastre cuando se quieren gestionar los registros DNS de cada dominio.

En busca de una interfaz y forma más sencilla de administrar estos registros (y tal vez los mismos dominios) es que se va a empezar a usar DNSimple para gestionar los registros DNS de los dominios:

- franciscoquintero.com
- devaspros.com
- enlacito.co

En este documento quedará registrado cada registro de todos los dominios y la forma de crearlos en DNSimple.

## Registros DNS

### franciscoquintero.com

#### Registro A ❌

```bash
dig +short A franciscoquintero.com
162.255.119.64
```

> [!Important]
> ¿Por qué sale este registro si no está creado por mí?
>
> Es por Github Pages y el URL Redirect que hay de `franciscoquintero.com` a `www.franciscoquintero.com`.
>
> Dado a que el subdominio `www` es el que apunta a la página en github pages (`cesc1989.github.io`).
>
> Para que funcione el Redirect Namecheap crea un registro A a la IP.

#### Registro TXT ❌

> [!Note]
> Esto existe porque está configurado el Email Forwarding. Al desactivarlo se borra y se hace innecesario copiarlo a DNSimple.

```bash
dig +short franciscoquintero.com TXT
"v=spf1 include:spf.efwd.registrar-servers.com ~all"
```

#### Registros CNAME

> [!Important]
> Sobre los CNAMEs
> 
> Hacer esto:
> ```bash
> dig +short franciscoquintero.com CNAME
> ```
>
> No devuelve nada porque los registros CNAME no se asigna, normalmente, a la raíz del dominio (`@`) sino a subdominio.

```bash
dig +short www.franciscoquintero.com CNAME
cesc1989.github.io.
```

```bash
dig +short lecturas.franciscoquintero.com CNAME
coshiapuntes.pikapod.net.
```


### devaspros.com

#### Registro A

```bash
dig +short cashflow.devaspros.com A
139.144.196.192

dig +short coshinotes.devaspros.com A
139.144.196.192

dig +short supermenu.devaspros.com A
134.195.90.227

dig +short postlane.devaspros.com A
134.195.90.227
```

#### Registro MX

```bash
dig +short send.resend.devaspros.com MX
30 feedback-smtp.sa-east-1.amazonses.com.
```

#### Registro CNAME

```bash
dig +short www.devaspros.com CNAME
devaspros.github.io.

dig +short intrati.devaspros.com CNAME
alias.zeit.co.
```

#### Registro TXT

```bash
dig +short devaspros.com TXT
"google-site-verification=xWs0c4dQuCReev6mRwM3K3DwD7LBq3hmCLt-Rnlirto"

dig +short send.resend.devaspros.com TXT
"v=spf1 include:amazonses.com ~all"
```

### enlacito.co

```bash

```

## Enlaces

- Configuración de dominio o subdominio en Github Pages: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#dns-records-for-your-custom-domain
- TTL en DNSimple: https://support.dnsimple.com/articles/what-is-ttl/
- Revisar respuesta WHOIS luego de configurar los DNS: https://support.dnsimple.com/articles/domain-resolution-issues/#check-name-server-list-in-the-whois-response