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

#### Registro A

```bash
dig +short A franciscoquintero.com
162.255.119.64
```

> [!Important]
> ¿Por qué sale este registro si no está creado por mí?
>
> Es por Github Pages y el redirect que hay de `franciscoquintero.com` a `www.franciscoquintero.com`.
>
> Dado a que el subdominio `www` es el que apunta a la página en github pages (`cesc1989.github.io`).

#### Registro TXT

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

#### URL Redirect

Hay una:
```
host Value
@    http://www.franciscoquintero.com unmasked
```

### devaspros.com

### enlacito.co


