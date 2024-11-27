# Deshaciendo cambios con git reset: —hard o --mix

Etiquetas: #git_reset

Tenía que deshacer varios cambios de la rama del code review luego de que esos cambios no iban a ser posible incluirlos en el repo.

Ya tenía varios commits entre el cambio a quitar y lo más reciente. No era práctico deshacerlo manualmente. Tenía que usar `git reset --hard`.

# Usando git reset —hard

> [!Note]
> git reset --hard deshace los cambios totalmente. Es decir, retorna el archivo a como estaba antes de haberlo modificado.
> 
> Es una acción destructiva. Se pierde todo.

La clave está en la cantidad de commits a deshace para llegar al punto deseado.

Para eso me tocó dejar la rama `ewa_14_exercises_web_client_pristine` me tocó dejarla como la había creado anteriormente mediante:
```
git reset --hard HEAD~4
```

> Nota: antes hice git show a todos los commits que había hecho y los guardé en archivos .txt
```
git reset --hard HEAD~4
HEAD is now at b497fdc3f EWA-14 Exercises via Web UI and configs
```

De esta forma, con la rama ya en el estado anterior, lo que hice fue aplicar los parches de los commits que quité y pude quitar los archivos de migración y los cambio de schema.

Usando `git apply` se introdujeron los cambios.

Recursos:

- Git apply -> [https://git-scm.com/docs/git-apply](https://git-scm.com/docs/git-apply)
- How to apply a patch generated with git format-patch? -> [https://stackoverflow.com/a/2250170/1407371](https://stackoverflow.com/a/2250170/1407371)

## Parcheando con git apply

*git apply --stat*

> Instead of applying the patch, output diffstat for the input. Turns off "apply".

```
git apply --stat ~/Downloads/frontendchange.txt
client/bundles/ExercisesViaWeb/services/api.ts |   12 ++++++------
1 file changed, 6 insertions(+), 6 deletions(-)
```

*git apply --check*

> Instead of applying the patch, see if the patch is applicable to the current working tree and/or the index file and detects errors. Turns off "apply".

```
git apply --check ~/Downloads/frontendchange.txt
```

# Usando git reset --mix

[!Note]
> git reset --mix deshace los cambios de manera parcial. Es decir, retorna el archivo a como estaba al haberlo modificado pero justo antes de haber hecho commit.
> 
> Permite quitar el archivo del "stage area" o hacer nuevos cambios sin perder lo que se haya hecho ya.

El comando es:
```
git reset --mix HEAD~NUM_COMMITS
```

Donde reemplazamos `NUM_COMMITS` por la cantidad de commits que queremos ir hacía atrás.

> Aquí idealmente querré hacer solo 1 commit hacía atrás para más control.

## Ejemplo de git reset --mix

Corriendo el comando:
```bash
$ git reset --mix HEAD~1
Unstaged changes after reset:
D       app/graphql/mutations/add_draft_care_plan.rb
D       app/services/incoming_faxes/draft_care_plan_assignment_service.rb
```

Revisando el estado de los archivos:
```bash
$ gst
On branch remove_draft_care_plan_assignment
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    app/graphql/mutations/add_draft_care_plan.rb
        deleted:    app/services/incoming_faxes/draft_care_plan_assignment_service.rb

no changes added to commit (use "git add" and/or "git commit -a")
```