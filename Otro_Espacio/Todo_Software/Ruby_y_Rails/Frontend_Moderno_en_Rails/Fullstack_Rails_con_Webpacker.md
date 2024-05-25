# Fullstack Rails con Webpacker
La necesidad de aprender [Webpacker](https://github.com/rails/webpacker) nace de ver cómo los proyectos fullstack en Rails que usan el *asset pipeline* y *sprockets* tienden a complicarse (archivos JavaScript y Stylesheets) con el tiempo.

Usando Webpacker, se puede aprovechar de mejor forma todos esos avances que ha habido en el mundo del frontend para hacer mejores apps fullstack con Ruby on Rails.

## Artículos sobre pasarse a Webpacker
- [Introducing Webpacker](https://medium.com/statuscode/introducing-webpacker-7136d66cddfb)
- [Goodbye Sprockets. Welcome Webpacker](https://medium.com/@coorasse/goodbye-sprockets-welcome-webpacker-3-0-ff877fb8fa79)
- [Using Webpacker in Ruby on Rails applications](https://www.lugolabs.com/articles/using-webpacker-in-ruby-on-rails-applications)
- [Evil Front Part 1: Modern Front-end in Rails](https://evilmartians.com/chronicles/evil-front-part-1)
    - [Evil Front Part 2, helper para partials ERB en carpeta](https://evilmartians.com/chronicles/evil-front-part-2) `[frontend](https://evilmartians.com/chronicles/evil-front-part-2)`
    - [Evil Front Part 3, mini tutorial de Action Cable con Webpacker](https://evilmartians.com/chronicles/evil-front-part-3)
## Artículo en contra de pasarse a Webpacker
- [Rails with Webpack - not for everyone](https://www.codementor.io/@help/rails-with-webpack-not-for-everyone-feucqq83z)
## Proyectos que Moví de Sprockets a Webpacker
- Feed & Fit, [Rama Webpackering](https://bitbucket.org/devaspros/feed-fit/branch/webpackering).
- [Webpackering Me](https://github.com/cesc1989/webpackering-me/)
# Apuntes de lo Aprendido en el Proceso
## Inicialmente

Se inicia una app con webpacker con `rails new app --webpacker` o instalando la gema.

**Comandos**
Se puede ver la lista de comandos que trae webpacker con `rails webpacker`

## Folder structure

Take a look at the new **app/*javascript** folder. This is the folder that will contain all of the JavaScript app code and webpack entry points (a.k.a packs). By default it’s `app/javascript/packs`

The convention here is that all webpack entry points should be placed in the `app/``*javascript/*``packs` folder and the modules can be placed inside `app/*javascript` folder in any way you like.


## Configuración de jQuery usando Webpacker
    const { environment } = require('@rails/webpacker')
    
    const webpack = require('webpack')
    
    environment.plugins.append('Provide', new webpack.ProvidePlugin({
      $: 'jquery',
      jQuery: 'jquery'
    }))
    
    module.exports = environment


## Otros Enlaces

Repo de [Go Rails](https://github.com/gorails-screencasts/bootstap-with-webpack-4) con configuración de Webpacker para guiarme.

En [este gist](https://gist.github.com/andyyou/834e82f5723fec9d2dc021fb7b819517), el tipo pone ejemplos de como configurar Postgres para columnas JSON y soportar UUID. Gist con configuraciones para Webpacker.

## Dando peso a la decisión de entender Webpacker

Por esto:

> If we are not careful enough with “classic Rails full-stack way”, we end up with the global dumpster of all things CSS and JS, littered with dead code, in no time.

Es que estoy estudiando Webpacker. En los proyectos Avanzza, Feed & Fit y Lupita, hay un reguero de JS y CSS bien horribles.

## Lecciones al migrar los JS de Feed & Fit a Webpacker
- 1er problema es cómo mover el jQuery de Nested form(viene integrado a la gema)
- Fue mejor mover toda la carpeta `app/assets/javascripts/adminlte` para poder hacerlo paso a paso
- Las funciones globales o que pertenecen a `window`, ejemplo: `const showTargetSize()` no las reconocían los archivos que las usaban aún estando importadas en el pack `application.js`. La solución fue meterlas en el objeto global `FeedAndFit` y usarlas accediendo mediante este `FeedAndFit.showTargetSize()`
## Sobre Algunos módulos antiguos y MomentJS

Ya importé la mayoría de los módulos excepto por *bootstrap-toggle* que da error de resolución.
Moment.js, fue otro que dio problema y parece que toca importarlo en cada archivo JS que lo usa.

Ver issue https://github.com/moment/moment/issues/2608 para un poco de contexto al respecto. [Esta respuesta](https://github.com/moment/moment/issues/2608#issuecomment-419793802) da a entender que tiene que estar importado en el contexto del código que lo ejecuta.

## Avances Finales

Se importaron todos los módulos desde NPM y no por archivo local. El error con boostrap-toggle es porque el `package.json` del repo no indica un archivo en el atributo *main*. Ver https://docs.npmjs.com/files/package.json#main

Ayer, [publiqué nested_form como paquete en NPM](https://www.npmjs.com/package/jquery_nested_form) para poder tener el archivo jquery_nested_form usando ES6:


- [Acá están las indicaciones](https://docs.npmjs.com/packages-and-modules/contributing-packages-to-the-registry) para publicar un paquete a NPM


# Assets y stylesheet_pack_tag

En Luxe, los estilos no recargaban solos al usar stylesheet_link_tag.

Se pasó a stylesheet_pack_tag y se recargaban automaticamente al haber cambios.

Enlaces sobre assets en Webpacker:

- CSS, Sass and SCSS https://github.com/rails/webpacker/blob/5-x-stable/docs/css.md
- Paths resolved https://github.com/rails/webpacker/tree/5-x-stable#resolved

