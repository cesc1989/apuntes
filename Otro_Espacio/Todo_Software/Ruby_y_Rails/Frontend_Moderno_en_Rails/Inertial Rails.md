# Probando Inertia Rails

Para hacer SuperMenu/Yummy, al querer integrar al Chino en este proyecto creo que se puede hacer una división natural entre el frontend y el backend sin necesidad de tener un repositorio por aparte para el frontend.

Esta separación la podemos lograr usando Inertia. Con Inertia podemos usar todo el poder de Rails + Hotwire para el Back office y dejar Rails + Inertia para el menú digital de cara al usuario.

## Instalación

A una aplicación Rails generada con Puntapie le instalé estas dos gemas:
```ruby
gem "inertia_rails", "~> 3.10"
gem "vite_rails", "~> 3.0"
```

Y luego ejecuté el comando del generador:
```ruby
bin/rails generate inertia:install
```

Le dije que quería instalar Vite Ruby, TailwindCSS y quería React:
```bash
Installing Inertia's Rails adapter
Could not find a package.json file to install Inertia to.
Would you like to install Vite Ruby? (y/n) y
         run  bundle add vite_rails from "."
Vite Rails gem successfully installed
         run  bundle exec vite install from "."
Vite Rails successfully installed
Would you like to use TypeScript? (y/n) n
Would you like to install Tailwind CSS? (y/n) y
Installing Tailwind CSS
         run  npm add tailwindcss @tailwindcss/vite @tailwindcss/forms @tailwindcss/typography --silent from "."
     prepend  vite.config.ts
      insert  vite.config.ts
      create  app/javascript/entrypoints/application.css
Adding Tailwind CSS to the application layout
      insert  app/views/layouts/application.html.erb
Adding Inertia's Rails adapter initializer
      create  config/initializers/inertia_rails.rb
Installing Inertia npm packages
What framework do you want to use with Inertia? [react, vue, svelte4, svelte] (react)
         run  npm add @inertiajs/react@latest @vitejs/plugin-react react react-dom --silent from "."
Adding Vite plugin for react
      insert  vite.config.ts
     prepend  vite.config.ts
Copying inertia.js entrypoint
      create  app/javascript/entrypoints/inertia.js
Adding inertia.js script tag to the application layout
      insert  app/views/layouts/application.html.erb
Adding Vite React Refresh tag to the application layout
      insert  app/views/layouts/application.html.erb
        gsub  app/views/layouts/application.html.erb
Copying example Inertia controller
      create  app/controllers/inertia_example_controller.rb
Adding a route for the example Inertia controller
       route  get 'inertia-example', to: 'inertia_example#index'
Copying page assets
      create  app/javascript/pages/InertiaExample.module.css
      create  app/javascript/assets/react.svg
      create  app/javascript/assets/inertia.svg
      create  app/javascript/assets/vite_ruby.svg
      create  app/javascript/pages/InertiaExample.jsx
Copying bin/dev
      create  bin/dev
Inertia's Rails adapter successfully installed
```

Con eso dado, al ejecutar el Rails server, fui a la página `localhost:3006/inertia-example` y cargó un componente de prueba que se auto generó.

# Detalles

Estas son todas las cosas que agrega el generador.

## Configuraciones de Vite

Agregó estos archivos:
```bash
bin/dev
bin/vite
config/vite.json
package-lock.json
package.json
vite.config.ts
```

### `bin/dev`

Es un archivo para correr el servidor Rails mediante Overmind, Hivemind o, finalmente, Foreman.

> [!Note]
> En mí caso siempre he preferido Foreman así que este archivo lo obvio.

### `bin/vite`

Este es el runner de Vite en Rails. Para ejecutarse Inertial-Rails lo incluyó en el archivo `Procfile.dev`:
```bash
vite: bin/vite dev
```

> [!Note]
> Toca tener una versión de Node reciente para que no explote al ejecutarse.

### `config/vite.json`

Incluye configuraciones de Vite. Ejemplo incluye la ubicación donde estará el código JS:
```json
{
  "all": {
    "sourceCodeDir": "app/javascript",
    "watchAdditionalPaths": []
  }
}
```

### Archivos package.json

Estos son los de siempre. Para indicar los paquetes JS a instalar.

Esto fue todo lo que instaló de salida:
```json
{
  "private": true,
  "type": "module",
  "devDependencies": {
    "vite": "^5.4.19",
    "vite-plugin-ruby": "^5.1.1"
  },
  "dependencies": {
    "@inertiajs/react": "^2.0.17",
    "@tailwindcss/forms": "^0.5.10",
    "@tailwindcss/typography": "^0.5.16",
    "@tailwindcss/vite": "^4.1.11",
    "@vitejs/plugin-react": "^5.0.0",
    "react": "^19.1.1",
    "react-dom": "^19.1.1",
    "tailwindcss": "^4.1.11"
  }
}
```

### `vite.config.ts`

Este no sé para qué es. Queda pendiente averiguar.

## Configuraciones de Inertia Rails

Por alguna razón hizo unas modificaciones en `config/initializers/content_security_policy.rb`. Eso aún no le doy uso en Rails así que ni idea.

Lo único que agregó fue el archivo `config/initializers/inertia_rails.rb`.

Ver docs: [Global Configuration](https://inertia-rails.dev/guide/configuration)

