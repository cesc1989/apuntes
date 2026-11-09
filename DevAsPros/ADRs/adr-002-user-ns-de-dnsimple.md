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

```bash
dig +short A franciscoquintero.com
162.255.119.64
```

### devaspros.com

### enlacito.co


