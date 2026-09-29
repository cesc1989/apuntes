# Apuntes Variados de las Guías Oficiales

## Cómo insertar nuevos registros de una Relation

```ruby
books_relation = app["relations.books"]
books_relation.insert(title: "Test Driven Development", author: "Kent Beck")
```

## Sobre los assets

El build es con [esbuild](https://esbuild.github.io/).

Comandos:
- `hanami assets compile`: los prepara para producción
- `hanami assets watch`: para en desarrollo compilar al cambiarlos

### Los Entrypoints

El compilado depende de los entrypoints:
> When Hanami compiles your assets, it detects your JavaScript **entry points**

Definición:
> An **entry point** is a file that serves as the starting point for a compiled **asset bundle**

El entrypoint por defecto es `app/assets/js/app.js`. Viene así al arrancar una app nueva:
```js
import "../css/app.css";
```

> [!Warning]
> Q: ¿Hay que incluir siempre el CSS en el entrypoint?
> A: tbc


> [!Importante]
> La guía dice:
> Solo los archivos JS y CSS referenciados en un entrypoint serán incluídos en el bundle luego de compilar.


#### Múltiples Entrypoints

La guía dice:
> You can have as many entry points as you like. You might create dedicated entry points for certain pages or features in your app, to improve page loading and rendering.

Así se pueden tener entrypoints para login, landing, admin, etc.


#### Crear un nuevo entrypoint

Se crean dentro de la carpeta `assets/js`. Se crea una nueva carpeta y luego el archivo debe ser `app.js`.

Ejemplo para login: `app/assets/js/login/app.js`. Y lo que podría haber:
```js
import "../../css/login/app.css";
import { resetPassword } from "./resetPassword";
```

Ejemplo de la app con dos entrypoints. El por defecto y el de login:
```bash
app/assets
├── css
│   ├── app.css
│   └── login
│       └── app.css
├── images
│   └── favicon.ico
└── js
    ├── app.js # Entry point
    └── login
        ├── app.js # Entry point
        └── resetPassword.js
```


### Usar los assets

Se pueden acceder en las _views_ o mediante el _assets component_.

El _assets component_ es un objeto registrado en `"assets"` la app (o el Slice).

En una consola de hanami:
```
app["assets"]
#<Hanami::Assets:0x0000000124856928
 @config=#<Hanami::Config::Assets:0x0000000123cb0f60 @__config__=#<Dry::Configurable::Config values={serve: true}>, @base_config=#<Hanami::Assets::Config:0x0000000123c7bd38 @__config__=#<Dry::Configurable::Config values={node_command: "node", path_prefix: "/assets", subresource_integrity: [], base_url: #<Hanami::Assets::BaseUrl:0x0000000124851e28 @url="">}>>>,
 @root=#<Pathname:/Users/francisco/projects/hanami-test/bookshelf/public/assets>>

Hanami.app["assets"]
#<Hanami::Assets:0x0000000124856928
 @config=#<Hanami::Config::Assets:0x0000000123cb0f60 @__config__=#<Dry::Configurable::Config values={serve: true}>, @base_config=#<Hanami::Assets::Config:0x0000000123c7bd38 @__config__=#<Dry::Configurable::Config values={node_command: "node", path_prefix: "/assets", subresource_integrity: [], base_url: #<Hanami::Assets::BaseUrl:0x0000000124851e28 @url="">}>>>,
 @root=#<Pathname:/Users/francisco/projects/hanami-test/bookshelf/public/assets>>

app["assets"]["app.js"]
#<Hanami::Assets::Asset:0x0000000123efdef8 @base_url=#<Hanami::Assets::BaseUrl:0x0000000124851e28 @url="">, @path="/assets/app.js", @sri=nil>
```


#### Helpers

También tiene los similares que hay en Rails:

- `asset_url`
- `javascript_tag`
- `stylesheet_tag`
- `image_tag`
- `favicon_tag`
- `video_tag`
- `audio_tag`


## Sobre Containers y Componentes

Esto parece ser un tema clave en Hanami.