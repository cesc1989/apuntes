# Plantilla de Rails apps 📰 
Por debajo, Rails usa [Thor](https://github.com/erikhuda/thor) para que funcionen los generadores y plantillas. Algunos métodos:

- inject_into_file ↔ insert_into_file
- [template](https://www.rubydoc.info/github/wycats/thor/Thor/Actions#template-instance_method)
- [directory](https://stackoverflow.com/a/7370011/1407371)
- copy_file ([cómo forzar copia](https://stackoverflow.com/questions/17992615/forcing-a-copy-with-a-thor-action))
    copy_file "file_name", force: true

Estos son generadores de Rails

- [environment](https://edgeapi.rubyonrails.org/classes/Rails/Generators/Actions.html#method-i-environment)
- [generate](https://edgeapi.rubyonrails.org/classes/Rails/Generators/Actions.html#method-i-generate)
- [rails_command](https://edgeapi.rubyonrails.org/classes/Rails/Generators/Actions.html#method-i-rails_command)
## Sobre Cómo Hacerlas
- [Rails Application Templates](https://guides.rubyonrails.org/rails_application_templates.html)
- [Artículo sobre template](https://dev.to/justalever/a-guide-to-using-ruby-on-rails-application-templates-12jh)
- [Usando](https://medium.com/@nahrivera7/personalizando-rails-new-e1bd3c44de4) `[.railsrc](https://medium.com/@nahrivera7/personalizando-rails-new-e1bd3c44de4)`
## Plantillas Existentes para Ver o Forkear

[Shareable Rails App Templates](https://www.reddit.com/r/rails/comments/glljbs/shareable_rails_app_templates/)

    - [Rails Bytes](https://railsbytes.com/)

**Sencillos**

- [Jumpstart](https://github.com/excid3/jumpstart)
- [Kickoff](https://github.com/justalever/kickoff_tailwind)
- [Sjablloon](https://github.com/frankwrk/sjabloon-lite)
- [Rails Template](https://github.com/mattbrictson/rails-template)

**Pagos**

- [The Ruby on Rails SaaS Template](https://bullettrain.co/)

**Olvidados**

- [Generapp de Koombea](https://github.com/koombea/generapp)

**Complejos**

- [Suspenders](https://github.com/thoughtbot/suspenders)

