# Revisando Solidus - eCommerce

> [!Important]
> En macos 13 no pude instalar vips. Me tocó usar Linux Mint para poder crear un nuevo Rails e instalar Solidus para poder probarlo.

## vips o imagemagick en Macos

### libvips

Repo: https://github.com/libvips/libvips

Para instalar vips
```
brew install vips
```

Para verificar:
```
vips --version
```

### imagemagick

Para verificar:
```
identify --version
```

## Sobrescribe algunos archivos

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

# Errores en Macos 🍏

## Could not open library 'vips.42': dlopen(vips.42, 0x0005): tried: 'vips.42' (no such file)

```
LoadError (Could not open library 'vips.42': dlopen(vips.42, 0x0005): tried: 'vips.42' (no such file), '/System/Volumes/Preboot/Cryptexes/OSvips.42' (no such file), '/usr/lib/vips.42' (no such file, not in dyld cache), 'vips.42' (no such file), '/usr/local/lib/vips.42' (no such file), '/usr/lib/vips.42' (no such file, not in dyld cache).
Could not open library 'libvips.42.dylib': dlopen(libvips.42.dylib, 0x0005): tried: 'libvips.42.dylib' (no such file), '/System/Volumes/Preboot/Cryptexes/OSlibvips.42.dylib' (no such file), '/usr/lib/libvips.42.dylib' (no such file, not in dyld cache), 'libvips.42.dylib' (no such file), '/usr/local/lib/libvips.42.dylib' (no such file), '/usr/lib/libvips.42.dylib' (no such file, not in dyld cache).
Searched in <system library path>, /opt/homebrew/lib, /opt/local/lib, /usr/local/lib, /usr/lib):
  
app/components/image_component.rb:15:in 'ImageComponent#call'
app/views/products/_featured_product_card.html.erb:3
app/views/home/_featured_products.html.erb:4
app/views/home/index.html.erb:1
```

## ActiveStorage::Transformers::ImageProcessingTransformer::UnsupportedImageProcessingMethod (One or more of the provided transformation methods is not supported

```
ActionView::Template::Error (One or more of the provided transformation methods is not supported.):

Causes:
ActiveStorage::Transformers::ImageProcessingTransformer::UnsupportedImageProcessingMethod (One or more of the provided transformation methods is not supported.)
    1: <div class="relative bg-black overflow-hidden rounded-lg md:rounded-xl lg:rounded-2xl">
    2:   <% image = product.gallery.images.second || product.gallery.images.first %>
    3:   <%= render(ImageComponent.new(
    4:     image: image,
    5:     size: size,
    6:     class: 'w-full object-cover'
  
app/components/image_component.rb:15:in 'ImageComponent#call'
app/views/products/_featured_product_card.html.erb:3
app/views/home/_featured_products.html.erb:4
app/views/home/index.html.erb:1
```

## Enlaces Misc

- Hilo en Reddit de hace 2 años: https://www.reddit.com/r/rails/comments/1g73kem/solidus_ecommerce_community/
- Sobre error de vips: https://stackoverflow.com/questions/70849182/could-not-open-library-vips-42-could-not-open-library-libvips-42-dylib