# Deshaciendo cambios con git reset —hard
Tenía que deshacer varios cambios de la rama del code review luego de que esos cambios no iban a ser posible incluirlos en el repo.

Ya tenía varios commits entre el cambio a quitar y lo más reciente. No era práctico deshacerlo manualmente. Tenía que usar `git reset` `--``hard`.

# Usando git reset —hard

La clave está en la cantidad de commits a deshace para llegar al punto deseado.

Para eso me tocó dejar la rama `ewa_14_exercises_web_client_pristine` me tocó dejarla como la había creado anteriormente mediante:


    $ git reset --hard HEAD~4


> Nota: antes hice git show a todos los commits que había hecho y los guardé en archivos .txt


    $ git reset --hard HEAD~4
    HEAD is now at b497fdc3f EWA-14 Exercises via Web UI and configs

De esta forma, con la rama ya en el estado anterior, lo que hice fue aplicar los parches de los commits que quité y pude quitar los archivos de migración y los cambio de schema.

Usando `git apply` se introdujeron los cambios.

Recursos:

- Git apply -> [https://git-scm.com/docs/git-apply](https://git-scm.com/docs/git-apply)
- How to apply a patch generated with git format-patch? -> [https://stackoverflow.com/a/2250170/1407371](https://stackoverflow.com/a/2250170/1407371)
## Parcheando con git apply

*git apply --stat*

> Instead of applying the patch, output diffstat for the input. Turns off "apply".
    $ git apply --stat ~/Downloads/frontendchange.txt
     client/bundles/ExercisesViaWeb/services/api.ts |   12 ++++++------
     1 file changed, 6 insertions(+), 6 deletions(-)

*git apply --check*

> Instead of applying the patch, see if the patch is applicable to the current working tree and/or the index file and detects errors. Turns off "apply".
    $ git apply --check ~/Downloads/frontendchange.txt

