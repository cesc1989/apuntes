# Revisando Solidus - Ecommerce

## Sobre escribe algunos archivos

Pensé que solo lo hacía en Puntapie pero es algo general de la gema `solidus_starter_frontend`:

```bash
  installing      [solidus_starter_frontend] installing files
    Overwrite /Users/francisco/projects/devaspros-projects/parcerito/app/controllers/application_controller.rb? (enter "h" for help) [Ynaqdhm] d
      class ApplicationController < ActionController::Base
    -   # Only allow modern browsers supporting webp images, web push, badges, import maps, CSS nesting, and CSS :has.
    -   allow_browser versions: :modern
      end
    Retrying...
    Overwrite /Users/francisco/projects/devaspros-projects/parcerito/app/controllers/application_controller.rb? (enter "h" for help) [Ynaqdhm] y
    Overwrite /Users/francisco/projects/devaspros-projects/parcerito/app/javascript/controllers/index.js? (enter "h" for help) [Ynaqdhm] d
    - // Import and register all your controllers from the importmap via controllers/**/*_controller
    + // Import and register all your controllers from the importmap under controllers/*
    + 
      import { application } from "controllers/application"
    + 
    + // Eager load all controllers defined in the import map under controllers/**/*_controller
      import { eagerLoadControllersFrom } from "@hotwired/stimulus-loading"
      eagerLoadControllersFrom("controllers", application)
    + 
    + // Lazy load controllers as they appear in the DOM (remember not to preload controllers in import map!)
    + // import { lazyLoadControllersFrom } from "@hotwired/stimulus-loading"
    + // lazyLoadControllersFrom("controllers", application)
    Retrying...
    Overwrite /Users/francisco/projects/devaspros-projects/parcerito/app/javascript/controllers/index.js? (enter "h" for help) [Ynaqdhm] y
```

## Enlaces Misc

- Hilo en Reddit de hace 2 años: https://www.reddit.com/r/rails/comments/1g73kem/solidus_ecommerce_community/