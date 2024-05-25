# Haciendo rebase de Omega a rama de trabajo
Aquí describo cómo funciona eso de hacer rebase de la rama principal (master, main, omega) en la rama de trabajo cuando hay conflicto o la rama de trabajo se atrasa y se necesitan los cambios más recientes de la principal.

# El Caso

En la rama `ewa_4_create_patient_exercise_web_tokens_table` tenía unos cambios de crear una nueva migración. Esta migración modificaba el archivo `db/schema`.

En los días siguientes a mis cambios y PR, hubo otras migraciones que modificaron el archivo schema.rb. Mi rama tenía conflicto con eso. Tenía que arreglar el conflicto lo que significaba que mis cambios tenían que mezclarse con los que ya estaban en Omega.

# Los pasos

La forma más limpia de hacer esto es con git rebase en esta forma:

1. Hacer pull de Omega
    1. `git checkout omega`
    2. `git pull origin omega`
2. Ubicarme en la rama de trabajo
    1. `git checkout ewa_4_create_patient_exercise_web_tokens_table`
3. Hacer rebase de omega en la rama de trabajo
    1. `git rebase omega`
4. Resolver el conflicto y agregar los archivos
    1. `git add db/schema.rb`
5. Continuar el rebase
    1. `git rebase` `--``continue`
6. Guardar el commit
    1. se abre el editor y se guardan los cambios

Una vez esto terminado, los commits de mi rama se pondrán por delante de los que venían de Omega y mi rama quedará en buen estado y mis commits no se perderán.

# El paso a paso visual en consola
## Paso 2
    $ gco ewa_4_create_patient_exercise_web_tokens_table 
    Switched to branch 'ewa_4_create_patient_exercise_web_tokens_table'


## Paso 3
    $ git rebase omega
    Auto-merging db/schema.rb
    CONFLICT (content): Merge conflict in db/schema.rb
    error: could not apply 2e1871cf7... EWA-4 create PatientExerciseWebToken model
    hint: Resolve all conflicts manually, mark them as resolved with
    hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
    hint: You can instead skip this commit: run "git rebase --skip".
    hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
    Could not apply 2e1871cf7... EWA-4 create PatientExerciseWebToken model


## Paso 4
    $ ga db/schema.rb


## Paso 5
    $ git rebase --continue
    [detached HEAD 2fc88ca84] EWA-4 create PatientExerciseWebToken model
     4 files changed, 28 insertions(+)
     create mode 100644 app/models/patient_exercise_web_token.rb
     create mode 100644 db/migrate/20240131044828_create_patient_exercise_web_tokens.rb
     create mode 100644 spec/factories/patient_exercise_web_tokens.rb
    Successfully rebased and updated refs/heads/ewa_4_create_patient_exercise_web_tokens_table.


## Revisión de orden de commits
    $ glone -5
    49a1e0c98 (HEAD -> ewa_4_create_patient_exercise_web_tokens_table) EWA-4 unique indices in patient_exercises_web_tokens table
    3d3982706 EWA-4 PatientExerciseWebToken: fix rubocop
    2fc88ca84 EWA-4 create PatientExerciseWebToken model
    558bd664c (origin/omega, origin/beta, origin/HEAD, omega) Launchpad default value set to True (#9255)
    965494f64 Create UI to enable Therapist Launchpad (#9186)

Los commits con prefijo “EWA-4” son los que quería tener de primeros en el historial.

