# ADR-002 Usar NS de DNSimple

Namecheap ofrece la compra de dominios a un precio menor que varios proveedores. También ofrece muchos productos más. Uno de esos productos es Name Servers (NS) de manera gratuita.

Desde el admin de dominios se pueden configurar muchos aspectos incluidos la configuración de DNS. Si bien se pueden configurar todo sin problemas, la UI de Namecheap es muy cargada y llena de upsells. El negocio de Namecheap es dominios pero ofrecen muchas cosas más. Todo eso crea un lastre cuando se quieren gestionar los registros DNS de cada dominio.

En busca de una interfaz y forma más sencilla de administrar estos registros (y tal vez los mismos dominios) es que se va a empezar a usar DNSimple para gestionar los registros DNS de los dominios:

- franciscoquintero.com
- devaspros.com
- enlacito.co

En este documento quedará registrado cada registro de todos los dominios y la forma de crearlos en DNSimple.

## Registros DNS

### franciscoquintero.com 🟢

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

dig +short resend._domainkey.resend.devaspros.com TXT
"p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDs4UMBIJS5mmB/vrGvo/A/rv4YAiurJ2rzEw6jbY/V9xA4UxH6VoXEVvApxsQkD28Ox3wWeRVxI8iEY2fqnI4fpkz0e1/J8AORN0vYh1MusT+ckE2u0ucA2Q2feXCyuZ7RWZbO96x1KYGmRH9h0Mfky4WqBSLnzMHwhuob6aO3uwIDAQAB"
```

### enlacito.co

#### Registro A

```bash
dig +short enlacito.co A
139.144.196.192
```

#### Registro MX

```bash
dig +short send.resend.enlacito.co MX
10 feedback-smtp.us-east-1.amazonses.com.
```

#### Registro TXT

```bash
dig +short send.resend.enlacito.co TXT
"v=spf1 include:amazonses.com ~all"

dig +short resend._domainkey.resend.enlacito.co TXT
"p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC+ZnZyPahBwY/IWf1L8c9owagbjaHrymzJKnI2MdkspiHX7HX8uKt3vpyrHNpKUubbro0uKxSpgpzvnRekcM077gueLumQ1lfVl8wNpfKy/35GmmXi7MGqzaQdWMXyo7RcUEmnNsUXLeA611KvgM6DM2f5+xkR9rPpMYsh8a30oQIDAQAB"
```

## Cómo importar a DNSimple

La primera forma es copiarlos manualmente. Yendo de Namecheap a DNSimple creando los registros. La otra opción es más práctica. En DNSimple se pueden auto importar.

En el Record Editor está el dropdown "Import/Export". Ahí aparece la opción "Auto Import DNS". Al ir a esa página DNSimple hace su mejor esfuerzo por traer los registros obvios.

Si faltan más registros de los retornados, se pueden ingresar subdominios en el campo justo debajo y clicar en "Add". Esto buscará y cargara los registros DNS correspondientes.

Finalmente, se clica en "Import DNS Records".

## Verificaciones luego de importar

### franciscoquintero.com 🟢

- [x] Comando dig lista los nuevos NS

```bash
dig NS franciscoquintero.com

; <<>> DiG 9.10.6 <<>> NS franciscoquintero.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 42709
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 1

;; ANSWER SECTION:
franciscoquintero.com.  3600    IN      NS      ns1.dnsimple-edge.com.
franciscoquintero.com.  3600    IN      NS      ns2.dnsimple-edge.net.
franciscoquintero.com.  3600    IN      NS      ns3.dnsimple-edge.io.
franciscoquintero.com.  3600    IN      NS      ns4.dnsimple-edge.org.
```

- [x] Carga sitio de lecturas
- [x] Carga desde dominio raíz
- [x] Carga desde subdominio www

### devaspros.com

- [ ] Comando dig lista los nuevos NS

```bash
dig NS devaspros.com
```

- [ ] Carga cashflow
- [ ] Carga coshinotes
- [ ] Carga supermenu
- [ ] Carga postlane
- [ ] Carga intrati
- [ ] Carga sitio web desde dominio raíz
- [ ] Carga sitio web desde subdominio www
- [ ] Verificación de DNS en Resend sigue verde
- [ ] Verificación de propiedad en Google Search Console sigue correcta


### enlacito

- [ ] Comando dig lista los nuevos NS

```bash
dig NS enlacito.co
```



## Enlaces

- Configuración de dominio o subdominio en Github Pages: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#dns-records-for-your-custom-domain
- TTL en DNSimple: https://support.dnsimple.com/articles/what-is-ttl/
- Revisar respuesta WHOIS luego de configurar los DNS: https://support.dnsimple.com/articles/domain-resolution-issues/#check-name-server-list-in-the-whois-response