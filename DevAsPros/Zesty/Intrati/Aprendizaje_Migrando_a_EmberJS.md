# Aprendizaje Migrando a EmberJS

## Enlaces de Interés
- [Ember.js Octane vs Classic Cheat Sheet](https://ember-learn.github.io/ember-octane-vs-classic-cheat-sheet/) 🤩 
- [Proyecto con buenos datos sobre Ember Octane](https://github.com/mike-north/ember-octane-workshop/tree/master/notes). 👍 
- [Este es un buen slide con buenos ejemplos](https://slides.com/gokatz/ember-octane-ticket-manager/fullscreen). 📺 
- Algunos RFCs para leer y entender los contextos y trasfondos
    - [RFC de @tracked](https://github.com/emberjs/rfcs/blob/master/text/0410-tracked-properties.md)
    - [RFC de Glimmer Component](https://github.com/emberjs/rfcs/blob/master/text/0416-glimmer-components.md)
## ¿Cómo es que todo se enlaza?
- Una ruta en `app/router.js`
    - Si necesita traer datos externos, un enrutador (o *route handler*) en `app/routes/archivo.js`
- Un *template* en `app/templates/archivo.hbs`
    - ¿Por qué?: Ver abajo en *Relación de ruta y template*
- Un componente de presentación en `app/components/componente.hbs`
    - Si el componente tiene comportamiento, una clase en `app/components/component.js`
## Relación de ruta y template

Según el generador `ember generate route route-name`:

> route <name> <options...>
>         Generates a route and a template, and registers the route with the router.

Lo que significa que una página/ruta se compone de:

- Definición en `app/router.js`
- Enrutador en `app/routes/route-name-js`
- Template en `app/templates/template-name.hbs`

Y según los [docs de EmberJS](https://guides.emberjs.com/release/routing/defining-your-routes/#toc_basic-routes):

> Now, when the user visits `/about`, Ember will render the `about` template. Visiting `/favs` will render the `favorites` template.

Más ejemplos de los docs:

    Router.map(function() {
      this.route('blog-post', { path: '/blog-post' });
    });


> The route defined above will by default use the `blog-post.js` route handler, the `blog-post.hbs` template, and be referred to as `blog-post` in any `<LinkTo />` components.
## Ejemplo Página About

**Ruta en** `**router.js**`

    Router.map(function() {
      this.route('about');
    });

No necesita enrutador para traer datos o personalizar la carga.

**Template en** `**app/templates/about.hbs**`

    <Jumbo>
      <h2>About Super Rentals</h2>
      <p>
        The Super Rentals website is a delightful project created to explore Ember.
        By building a property rental site, we can simultaneously imagine traveling
        AND building Ember applications.
      </p>
      <LinkTo @route="contact" class="button">Contact Us</LinkTo>
    </Jumbo>

No necesita componente presentacional.

## Ejemplo Página Inicial

No tiene ruta en `router.js` pero sí tiene enrutador en `app/routes/index.js`

    import Route from '@ember/routing/route';
    
    const COMMUNITY_CATEGORIES = [
      'Condo',
      'Townhouse',
      'Apartment'
    ];
    
    export default class IndexRoute extends Route {
      async model() {
        let response = await fetch('/api/rentals.json');
        let { data } = await response.json();
    
        return data.map(model => {
          let { id, attributes } = model;
          let type;
    
          if (COMMUNITY_CATEGORIES.includes(attributes.category)) {
            type = 'Community';
          } else {
            type = 'Standalone';
          }
    
          return { id, type, ...attributes };
        });
      }
    }

**Template en** `**app/templates/index.hbs**`

    <Jumbo>
      <h2>Welcome to Super Rentals!</h2>
      <p>We hope you find exactly what you're looking for in a place to stay.</p>
      <LinkTo @route="about" class="button">About Us</LinkTo>
    </Jumbo>
    <div class="rentals">
      <ul class="results">
        {{#each @model as |rental|}}
          <li><Rental @rental={{rental}} /></li>
        {{/each}}
      </ul>
    </div>

No usa componente `index.hbs` pero sí usa el componente `Rental`.


## Error al Transicionar Mal a una Ruta

Cuando intentaba esto: `this.router.transitionTo('create-room', this.roomName)` desde una clase, obtuve este error:


> More context objects were passed than there are dynamic segments for the route:

Para corregirlo, tenía que hacer que la ruta soportora *dynamic segments* así:
`this.route('create-room', { path: '/rooms/:name' });`

**Relacionado**

- Documentación de `[transitionTo](https://api.emberjs.com/ember/release/classes/Route/methods/transitionTo?anchor=transitionTo)`
- Respuesta en [Stack Overflow](https://stackoverflow.com/a/17937467/1407371)
## Ciclo de vida del Componente y Element Modifiers

Resulta que en Ember Octane ya no se usan los métodos del ciclo de vida del componente. Ahora se favorecen los modificadores de elementos.

Para poder lanzar una función cuando el componente termina de renderizar se usa un modificador que se escribe en el componente con markup y no una función que va en la clase.


    // edit-form.hbs
    <form>
      <input {{did-insert this.focus}}>
    </form>
    
    // edit-form.js
    export default class EditForm {
      focus(element) {
        element.focus();
      }
    }

**Relacionado**

- [Ember Render Modifiers](https://github.com/emberjs/ember-render-modifiers)
- [Al Respecto en las guía](https://guides.emberjs.com/release/components/template-lifecycle-dom-and-modifiers/#toc_calling-methods-on-first-render)
- En este artículo se explica mejor el tema con respecto a los *life cycle hooks* y *element modifiers* [Ember Octane in 5 Minutes](https://jenweber.netlify.com/ember.js-octane-in-five-minutes/)
- Acá explican bien que hay que usar los *modifiers* y **marcarlos con _@action_**: [Ember Cheat Sheet](https://ember-learn.github.io/ember-octane-vs-classic-cheat-sheet/#section-using-didinsert)

