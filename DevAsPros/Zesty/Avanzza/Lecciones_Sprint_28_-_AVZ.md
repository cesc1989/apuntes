# Lecciones Sprint 28 - AVZ

## Plugins jQuery

Para recargar una lista de selección que usa *bootstrap selectpicker* puede mostrar los contenidos que cambien dinámicamente con la opción `['](https://developer.snapappointments.com/bootstrap-select/methods/#selectpickerrefresh)``[refresh](https://developer.snapappointments.com/bootstrap-select/methods/#selectpickerrefresh)``['](https://developer.snapappointments.com/bootstrap-select/methods/#selectpickerrefresh)`


    $('.selectpicker').selectpicker('refresh');

Este plugin también tiene la opción para seleccionar todos y/o deseleccionar todos con la opción `actionsBox`:


    $('#control_card_vehicle_ids').selectpicker({
      actionsBox: true
    })


## Double splat para Hash y ActionController::Parameters

El operador [*splat*](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=2ahUKEwiF1PWOwsjhAhXMpFkKHdhkBwUQFjAAegQIARAB&url=https%3A%2F%2Fwww.justinweiss.com%2Farticles%2Ffun-with-keyword-arguments%2F&usg=AOvVaw2iz6l3o2hBQx_N3dIrcYoM) sirve para obtener todos los valores de un Array sin tener que accederlos por índice:


    pry > a = [1, 2, 3]
    pry > p *a
    1
    2
    3
    => [1, 2, 3]

Del mismo modo, así como en JavaScript existe el *spread,* en Ruby tenemos [*double splat*](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&cad=rja&uact=8&ved=2ahUKEwiF1PWOwsjhAhXMpFkKHdhkBwUQFjABegQIABAB&url=https%3A%2F%2Fthepugautomatic.com%2F2017%2F02%2Fdouble-splat-to-merge-hashes%2F&usg=AOvVaw08dMSCiochjkg6s5i-XcQ6) para los Hash.


    pry > c = { a: 1, b: 2 }
    pry > p **c
    => {:a=>1, :b=>2}

Sin embargo, al intentar hacer eso en una instancia de `ActionController::Parameters` da un error:


    TypeError: wrong argument type String (expected Symbol)

Y ocurre porque la forma en que `ActionController::Parameters` exporta su instancia a un Hash es con las llaves como String y no como Symbol.


    {"hola"=>"mundo", "vaya"=>"vaya"}


## ActionController::Parameters#to_h o ActionController::Parameters#to_hash

La diferencia es que `#to_h` no transforma a hash las llaves no permitidas. En cambio `[to_hash](https://api.rubyonrails.org/classes/ActionController/Parameters.html#method-i-to_hash)` si transforma todas, así sea que no estén permitidas.


## Mailers y Rails

Hacer [vistas para correos](http://www.simonnordberg.com/creating-robust-email-templates-in-action-mailer/) son complicados porque todos los estilos tienen que ser aplicados en línea y la estructura hecha en tablas.

Usando la gema `[premailer-rails](https://github.com/fphilipe/premailer-rails)` se puede usar una hoja de estilo por separado y la gema se encargará de aplicar los estilos en línea. Haciendo este trabajo mucho más sencillo.


## Meses del año en letras y traducidos

Para obtener el mes del año se puede hacer de varias formas:

    Date::MONTHNAMES[Date.today.month]
    
    Date.today.strftime("%B")

Y se puede [tener traducida](https://stackoverflow.com/a/32377036/1407371) así:

    I18n.l(Time.current, format: "%B")


## has_many :through con inverse_of

Este es el caso donde intentaba crear una `control_card_request` en bache pero al intentar crear el objeto, arrojaba una validación de `Vehicles: is invalid` en la colección, posiblemente por lo explicado en [este hilo en Stack Overflow](https://stackoverflow.com/a/37400594/1407371).

Documentación al respecto con ejemplo: [Setting Inverses](https://api.rubyonrails.org/classes/ActiveRecord/Associations/ClassMethods.html#module-ActiveRecord::Associations::ClassMethods-label-Setting+Inverses)

