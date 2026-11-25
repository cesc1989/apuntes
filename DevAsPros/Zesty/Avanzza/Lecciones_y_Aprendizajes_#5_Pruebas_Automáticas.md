# Lecciones y Aprendizajes #5 Pruebas Automáticas

- Todos los tipos de tests en Rails 5: [https://mad.is/2016/12/rails-5-what-to-test-where/](https://mad.is/2016/12/rails-5-what-to-test-where/)
- Test de vistas en Rails con `assert_select` [https://guides.rubyonrails.org/testing.html#testing-views](https://guides.rubyonrails.org/testing.html#testing-views)
- `assert_select` docs: https://apidock.com/rails/ActionDispatch/Assertions/SelectorAssertions/assert_select



## Diferencia entre los *feature* y *request* specs de RSpec:
- Stack Overflow: https://stackoverflow.com/questions/15173946/rspec-what-is-the-difference-between-a-feature-and-a-request-spec
- Mike Williamsom: https://mikewilliamson.wordpress.com/2013/06/12/the-difference-between-feature-specs-and-request-specs/


## Pruebas de Vistas
- RSpec: https://github.com/rspec/rspec-rails#view-specs
- RSpec web oficial: https://relishapp.com/rspec/rspec-rails/docs/view-specs/view-spec
- En Thoughtbot: https://robots.thoughtbot.com/how-we-test-rails-applications#view-specs


## Error de Devise en las vistas:
> ActionView::Template::Error:
> 
> Devise could not find the `Warden::Proxy` instance on your request environment.
> Make sure that your application is loading Devise and Warden as expected and that the `Warden::Manager` middleware is present in your middleware stack.
> 
> If you are seeing this on one of your tests, ensure that your tests are either executing the Rails middleware stack or that your tests are using the `Devise::Test::ControllerHelpers` module to inject the `request.env['warden']` object for you.

En esta ocasión ocurría en tests de vistas por lo que tuve que cambiar `:controller` por `:view` y funcionó :)

    config.include Devise::Test::ControllerHelpers, type: :view

