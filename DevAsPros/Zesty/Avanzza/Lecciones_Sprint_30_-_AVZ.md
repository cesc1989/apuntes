# Lecciones Sprint 30 - AVZ

## Ejecutar otras rake tasks desde una rake task

Como indica [la respuesta seleccionada](https://stackoverflow.com/a/1290119/1407371)


    namespace :production_bootstrap do
      desc 'Setup production'
      task :run_all do
        Rake::Task['categories:run_all'].invoke
        Rake::Task['categories:create_for_suppliers'].invoke
        Rake::Task['infractions:create_types'].invoke
        Rake::Task['vehicles:run_all'].invoke
        Rake::Task['cities:create_basic_cities'].invoke
        Rake::Task['countries:create_countries'].invoke
        Rake::Task['admins_and_owners:create_admins'].invoke
        Rake::Task['admins_and_owners:create_owners'].invoke
      end
    end


## Formularios: disabled vs readonly

[Hay una gran diferencia](https://stackoverflow.com/questions/7357256/disabled-form-inputs-do-not-appear-in-the-request) entre un campo con el atributo `disabled` y con el atributo `readonly`.

Los desactivado, no se mandan en una petición `POST` al servidor. Los solo lectura sí.

Además, una lista de selección con el atributo `readonly` igual [se seguirá desplegando aunque esté en modo solo lectura](https://github.com/plataformatec/simple_form/issues/1513). Aunque al parecer [hay soluciones para esto](https://stackoverflow.com/questions/368813/html-form-readonly-select-tag-input) pero toca meter más JavaScript.


## rails db:schema:load vs rails db:migrate

La principal diferencia está en que `rails db:schema:load` [es destructivo cuando hay información en la base de datos](https://stackoverflow.com/a/5905958/1407371) **y solamente debería usarse la primera** vez que arranca un sistema con base de datos limpia.

De resto, siempre debe usarse `rails db:migrate`

