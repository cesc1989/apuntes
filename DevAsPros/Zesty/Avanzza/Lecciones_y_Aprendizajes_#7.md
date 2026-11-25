# Lecciones y Aprendizajes #7

- Como setear una lista de selección como multiple en simple form [https://stackoverflow.com/questions/22159614/ruby-on-rails-4-simple-form-multiple-select-input](https://stackoverflow.com/questions/22159614/ruby-on-rails-4-simple-form-multiple-select-input)
- desminificar en línea [https://unminify.com/](https://unminify.com/)
- Pasar array a string con `join` https://stackoverflow.com/questions/3500814/ruby-array-to-string-conversion
- Librería para el datepicker: https://github.com/T00rk/bootstrap-material-datetimepicker
- Validación unica en Rails [https://guides.rubyonrails.org/active_record_validations.html#uniqueness](https://guides.rubyonrails.org/active_record_validations.html#uniqueness)
- Iteración de Arrays con jQuery.each() [https://www.sitepoint.com/jquery-each-function-examples/](https://www.sitepoint.com/jquery-each-function-examples/)
- Iteración de Arrays nativamente en Javascript [https://www.digitalocean.com/community/tutorials/how-to-use-array-methods-in-javascript-iteration-methods](https://www.digitalocean.com/community/tutorials/how-to-use-array-methods-in-javascript-iteration-methods)



# Averiguar integración con WhatsApp

WhatsApp no ofrece ningún API para esto. Sin embargo, hay formas de lograrlo pero en dispositivos solamente, no en la web.

Según https://faq.whatsapp.com/en/android/28000012 se puede mediante estas dos formas, en Android:
Formato URL

    whatsapp://send?text=

O usando los Intents de Android.

También existe la forma *Click to chat* donde a unna URL en específico se le pasa el número de celular completo, en formato internacional.

Ejemplos según https://faq.whatsapp.com/en/general/26000030

    Use: https://wa.me/15551234567
    
    Don't use: https://wa.me/+001-(555)1234567

Visto también en Stack Overflow https://stackoverflow.com/questions/29218378/mobile-website-whatsapp-button-to-send-message-to-a-specific-number

Por último, también existe el API de WhatsApp Business pero se reservan el derecho de admisión :D

Ver https://www.whatsapp.com/business/api


# En rails 5.1 ahora es `form_with` en vez de `form_for` o `form_tag`

Ver:

- [Rails 5.1's form_with vs. form_tag vs. form_for](https://m.patrikonrails.com/rails-5-1s-form-with-vs-old-form-helpers-3a5f72a8c78a)
- [Documentacion](https://api.rubyonrails.org/classes/ActionView/Helpers/FormHelper.html#method-i-form_with)


# Error de token de autenticidad en proveedores

Ocurre cuando un proveedor responde a una cotización:

    Started PUT "/s/quotations/13" for 181.192.208.85 at 2018-12-28 19:32:30 +0000 
    Processing by Supplier::QuotationsController#update as HTML 
      Parameters: {"utf8"=>"✓", "authenticity_token"=>"wau18HOBAxzznhPGj5sAP3uH/Uvjodo1fTnuB+PWCD89C1wbdEk1Fnqv2byhA9cK5gjnJlbv2GzNm/FCfC9tcQ==", "supplier_answer"=>"Esféricas. ($25.865 c/u) $52.730\r\nBrazo Axial. $27.342\r\nTerminal. $23.436    \r\nBujes barra estabilizadora.  ($ 2.930 c/u) $5.860\r\nBandas. $37.106\r\nMuñecos colgantes.  ($ 13.670 c/u) $27.342\r\nValor Total. $173.815", "commit"=>"Enviar respuesta", "id"=>"13"} 
    Can't verify CSRF token authenticity. 
    Completed 422 Unprocessable Entity in 1ms (ActiveRecord: 0.0ms) 
    User excluded error: #<ActionController::InvalidAuthenticityToken: ActionController::InvalidAuthenticityToken> 
    
    ActionController::InvalidAuthenticityToken (ActionController::InvalidAuthenticityToken): 

La posible solución está dado por lo que acá aparece: [https://stackoverflow.com/a/43356164/1407371](https://stackoverflow.com/a/43356164/1407371)


# Selectores jQuery para encontrar elementos por patrón
- Encontrar grupo de elementos por patrón en el ID o clase: https://stackoverflow.com/questions/1487792/jquery-find-element-whose-id-has-a-particular-pattern
- Documentación de jQuery: https://api.jquery.com/category/selectors/
- Tabla ASCII de caracteres en HTML [https://www.ascii.cl/htmlcodes.htm](https://www.ascii.cl/htmlcodes.htm)
- Clausula guardia en JavaScript: [https://stackoverflow.com/questions/5339121/how-do-you-implement-a-guard-clause-in-javascript](https://stackoverflow.com/questions/5339121/how-do-you-implement-a-guard-clause-in-javascript)
- Cómo validar que un valor es `null` o `undefined` en JavaScript: https://stackoverflow.com/questions/5101948/javascript-checking-for-null-vs-undefined-and-difference-between-and
- Helper para formatear unidades de dinero: `[number_to_currency](https://api.rubyonrails.org/classes/ActionView/Helpers/NumberHelper.html#method-i-number_to_currency)`
- Grilla flexible con todas las columnas del mismo ancho con Bootstrap 4: [https://getbootstrap.com/docs/4.2/layout/grid/#equal-width](https://getbootstrap.com/docs/4.2/layout/grid/#equal-width)



# Floating Action Button

FAB para web https://codepen.io/SinceSidSlid/pen/vExJaP

> No me funcionó muy bien que digamos

Otro FAB en versión jQuery plugin: https://github.com/katrincwl/kc_fab

jQuery mousedown vs click [https://stackoverflow.com/questions/14839440/jquery-mousedown-vs-click](https://stackoverflow.com/questions/14839440/jquery-mousedown-vs-click)


- Ajax y jQuery para formularios en Rails 5 [https://rubyplus.com/articles/4211-Using-Ajax-and-jQuery-in-Rails-5-Apps](https://rubyplus.com/articles/4211-Using-Ajax-and-jQuery-in-Rails-5-Apps)
- Otra forma de tener formularios remotos con Rails 5 y usa ES6 [5 Steps to Add Remote Modals to Your Rails App](https://jtway.co/5-steps-to-add-remote-modals-to-your-rails-app-8c21213b4d0c)
- Cargar assets en archivos sass https://github.com/rails/sass-rails#asset-helpers
- Cómo añadir correctamente el código de google analytics a una aplicación Rails [https://richonrails.com/articles/adding-google-analytics-to-your-rails-app-the-right-way](https://richonrails.com/articles/adding-google-analytics-to-your-rails-app-the-right-way)



# Testing de Mailers
- Testeo de mailers en Rails 5 [https://guides.rubyonrails.org/testing.html#testing-your-mailers](https://guides.rubyonrails.org/testing.html#testing-your-mailers)
- gema email_spec para testear mailers https://github.com/email-spec/email-spec


# Propagación de registros DNS y Mailgun
- Para revisar la propagación de registros DNS [https://www.whatsmydns.net](https://www.whatsmydns.net)
- Revisión de la verificación de dominio en mailgun [https://help.mailgun.com/hc/en-us/articles/360011565514-Troubleshooting-DNS-records-to-verify-a-domain-](https://help.mailgun.com/hc/en-us/articles/360011565514-Troubleshooting-DNS-records-to-verify-a-domain-)
- Guía de integración de Mailgun con Namecheap: [https://www.namecheap.com/support/knowledgebase/article.aspx/10086/2208/how-to-connect-a-domain-to-mailgun](https://www.namecheap.com/support/knowledgebase/article.aspx/10086/2208/how-to-connect-a-domain-to-mailgun)
- ⭐ **Solución** https://medium.com/@ustcboulder/setup-mailgun-on-namecheap-spf-dkim-cname-and-mx-684b5c1fb492


# Cómo usar campos JSON o JSONB de Postgres en Rails
- [Codeship](https://blog.codeship.com/unleash-the-power-of-storing-json-in-postgres/)
- [Guías de Rails edge](https://edgeguides.rubyonrails.org/active_record_postgresql.html#json-and-jsonb)


# IIFE en JS
- https://stackoverflow.com/questions/8228281/what-is-the-function-construct-in-javascript
- IIFE con signo de admiración adelante https://stackoverflow.com/questions/3755606/what-does-the-exclamation-mark-do-before-the-function/5654929#5654929

Tiene un buen truco para impedir que ciertas funciones se ejecuten en páginas que no queremos: https://brandonhilkert.com/blog/organizing-javascript-in-rails-application-with-turbolinks/
Ejemplo en coffescript

    $(document).on "turbolinks:load", ->
      return unless $(".posts.index").length > 0
      f = new App.Chart $("#chart")
      f.render()


# Como hacer mejores consultas en Active Record usando Arel

[https://jacopretorius.net/2016/09/the-mimimum-arel-every-rails-developer-should-know.html](https://jacopretorius.net/2016/09/the-mimimum-arel-every-rails-developer-should-know.html)


# Cómo verificar si una variable está definida?

https://www.rubyguides.com/2018/10/defined-keyword/

- para variables locales: `local_variables.include?(:orange)`
- para variables de instancia: `instance_variable_defined?("@food")`

pero es mejor si se usa el operador de navegación segura de Ruby:

> Try the “safe navigator operator” (Ruby 2.3+) which only calls a method if the variable is not nil. @vehicle&.id`



- Cómo validar fechas en Rails [https://guides.rubyonrails.org/active_record_validations.html#custom-methods](https://guides.rubyonrails.org/active_record_validations.html#custom-methods)
- Gema para validar fechas [https://github.com/adzap/validates_timeliness/](https://github.com/adzap/validates_timeliness/)
- bootstrap material date time picker [http://t00rk.github.io/bootstrap-material-datetimepicker/](http://t00rk.github.io/bootstrap-material-datetimepicker/)
- Formas de parsear fechas con MomentJS [https://momentjs.com/docs/#/parsing/](https://momentjs.com/docs/#/parsing/)



