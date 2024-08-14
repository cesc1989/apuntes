# Temas de Git

# Autocompletar comandos con aliases de shell

- [How do I get bash completion to work with aliases?](https://stackoverflow.com/questions/342969/how-do-i-get-bash-completion-to-work-with-aliases)
- [Git branch bash autocomplete *with aliases* (add to .bash_profile)](https://gist.github.com/JuggoPop/10706934)
- [How can I get bash to perform tab-completion for my aliases?](https://superuser.com/questions/436314/how-can-i-get-bash-to-perform-tab-completion-for-my-aliases)


# Firmar Commits en Git

Lo mejor es generar la firma gpg con el comando

    $ gpg --full-generate-key

Va a pedir:

- El tipo: usar por defecto
- Tamaño de bits: usar 4096
- Ingresa nombre, correo
- Passprhase para hacerla segura

Para listar las llaves existentes se pueden usar

    $ gpg --list-secret-keys --keyid-format LONG
    /Users/fquintero/.gnupg/pubring.kbx
    -----------------------------------
    sec   rsa4096/6B6E024D8276BC8A 2020-06-26 \[SC\] [expires: 2023-06-26]
          7491AD1E39BF1D080A66C9176B6E024D8276BC8A
    uid                 [ultimate] Francisco Quintero Coronell <frajaquico@yahoo.es>
    ssb   rsa4096/71060385A7D4DC69 2020-06-26 \[E\] [expires: 2023-06-26]
    
    $ gpg --list-secret-keys --keyid-format SHORT
    /Users/fquintero/.gnupg/pubring.kbx
    -----------------------------------
    sec   rsa4096/8276BC8A 2020-06-26 \[SC\] [expires: 2023-06-26]
          7491AD1E39BF1D080A66C9176B6E024D8276BC8A
    uid         [ultimate] Francisco Quintero Coronell <frajaquico@yahoo.es>
    ssb   rsa4096/A7D4DC69 2020-06-26 \[E\] [expires: 2023-06-26]

Se puede verificar que el agente gpg esté funcionando con:

    $  echo "hello world" | gpg --clearsign
    -----BEGIN PGP SIGNED MESSAGE-----
    Hash: SHA256
    
    hello world
    gpg: signing failed: Inappropriate ioctl for device
    gpg: [stdin]: clear-sign failed: Inappropriate ioctl for device

El cual se puede reiniciar así:

    $ gpgconf --kill gpg-agent
    $ gpgconf --launch gpg-agent

Enlaces relacionados:

- https://help.github.com/en/github/authenticating-to-github/telling-git-about-your-signing-key
- [How (and why) to sign Git commits](https://withblue.ink/2020/05/17/how-and-why-to-sign-git-commits.html)
- Cómo remover llaves existentes → http://blog.chapagain.com.np/gpg-remove-keys-from-your-public-keyring/

**Error de gpg**
Cuando quería firmar los commits, salía este error:

    $ error: gpg failed to sign the data
    $ fatal: failed to write commit object

Se soluciona muy fácil pero tomó mucho tiempo darme cuenta:

    $ export GPG_TTY=$(tty)

Visto en:

- https://github.com/keybase/keybase-issues/issues/2798
- En [esta pregunta](https://stackoverflow.com/questions/39494631/gpg-failed-to-sign-the-data-fatal-failed-to-write-commit-object-git-2-10-0) hay varias opciones que no me funcionaron

**Copiar la llave gpg en GitHub**
Primero hay que listar las llaves gpg y copiar el ID de la propia:

    $ gpg --list-secret-keys --keyid-format LONG
    /Users/fquintero/.gnupg/pubring.kbx
    -----------------------------------
    sec   rsa4096/6B6E024D8276BC8A 2020-06-26 \[SC\] [expires: 2023-06-26]
          7491AD1E39BF1D080A66C9176B6E024D8276BC8A
    uid                 [ultimate] Francisco Quintero Coronell <frajaquico@yahoo.es>
    ssb   rsa4096/71060385A7D4DC69 2020-06-26 \[E\] [expires: 2023-06-26]

Luego, se envuelve la llave en una armadura:

    $ gpg --armor --export 6B6E024D8276BC8A
    -----BEGIN PGP PUBLIC KEY BLOCK-----
    ...
    -----END PGP PUBLIC KEY BLOCK-----

Se copia todo el contenido generado y se lleva a GitHub (sección de configuraciones “SSH and GPG keys”) y listo!

![](https://paper-attachments.dropbox.com/s_F1CE871F7ED9A72D4681536E321A7DE52CDD5D8E87B5BE57118EB75C8D763D34_1593205882019_image.png)

Ver → https://help.github.com/en/github/authenticating-to-github/generating-a-new-gpg-key

# Cómo Remover llaves gpg existentes

Este comando `gpg --delete-key key-``**ID**` sirve pero primero toca borrar los *secrets*.

    $ gpg --delete-key A2CCE97B
    
    $ gpg: there is a secret key for public key "A2CCE97B"!
    $ gpg: use option "--delete-secret-keys" to delete it first.


# Cómo guardar el passphrase para que no lo pida gpg-agent a cada commit

Normalmente, cada commit el gpg-agent lo pedirá para firmar los commits. Pero lo podemos configurar para que sea requerido menos veces.

La clave está en el archivo `~/.gnupg/gpg-agent.conf`:

    default-cache-ttl 28800
    max-cache-ttl 28800

Si no existe, se crea y se guarda con esos valores.

Ver:

- [Remember GPG password when signing git commits](https://stackoverflow.com/questions/36847431/remember-gpg-password-when-signing-git-commits)
- [Gist sobre todo este asunto](https://gist.github.com/ankurk91/c4f0e23d76ef868b139f3c28bde057fc#make-gpg-remember-your-passphrase-tricky)

# Condicionales en el gitconfig

Ver [Conditionally setting your gitconfig](https://utf9k.net/blog/conditional-gitconfig/) y [Can gitconfig options be set conditionally?](https://stackoverflow.com/questions/14754762/can-gitconfig-options-be-set-conditionally)

También acá [Conditional git profile configuration](https://dev.to/tastefulelk/conditional-git-profile-configuration-212b)

El cual según el comentario que puse, me sirvió mucho.

# Gitsieate

## Cómo funciona git reset?

[git reset explicado](https://git-scm.com/book/en/v2/Git-Tools-Reset-Demystified)


## Cómo cambiar nombre de rama?

Estando en la rama a la cual se le quiere cambiar el nombre:

    git branch -m newname

Visto en [Stack Overflow](https://stackoverflow.com/questions/3866951/change-a-branch-name-in-a-git-repo#3866972).

## Cómo revertir un commit?

Esto es lo que hace GitHub en la web cuando se revierte un PR. Es decir, crea un nuevo commit donde todos los cambios ya mezclados se deshacen.

Se hace con:

    git revert {commit_id}


> Reverting a commit means to create a new commit that undoes all changes that were made in the bad commit. Just like above, the bad commit remains there, but it no longer affects the the current master and any future commits on top of it.

Visto en [este gist](https://gist.github.com/gunjanpatel/18f9e4d1eb609597c50c2118e416e6a6).

## Cómo remover un commit con git rebase?

*Teniendo en cuenta que los cambios no se hayan mandado a ningún remoto.*

Se hace con el rebase interactivo e indicando la cantidad de commits a revisar para mezclar:

    # git rebase -i HEAD~N
    $ git rebase -i HEAD~5

Donde `HEAD~5` es la cantidad de commits a revisar para mezclar/rebasar.

Visto en [Stack Overflow](https://stackoverflow.com/questions/1338728/how-do-i-delete-a-commit-from-a-branch/1338758#1338758).

Otra posibilidad es así:

    git rebase -i <commit>~1

Según el autor:

> This will start the rebase in interactive mode `-i` at the point just before the commit you want to whack.

Visto en esta [otra respuesta](https://stackoverflow.com/a/1338756/1407371).

# Flujo de trabajo en Git: cuando un PR está pendiente y quiero seguir trabajando en la siguiente historia

> I'm working on a project and I submitted my first pull request and while I'm waiting I want to continue on working on my project building up from what I worked on the merge that's still pending on

La respuesta es amplia. Ver en [Stack Overflow](https://stackoverflow.com/questions/35790561/working-while-waiting-for-pending-pr).

El resumen sería:

1. Crear la 1era rama: *feature1*
2. Completa el trabajo y manda el PR
3. Crea la nueva rama a partir del trabajo anterior: *feature2* 
4. Cuando la 1era rama sea mezclada en master, mezclar la 2da no presentará problemas


# Cómo configurar git para no escribir usuario y contraseña cuando no se tiene llave SSH

En realidad, la mejor forma es con la llave SSH así que esto no lo recomendaría pero en todo caso queda el [enlace](https://stackoverflow.com/a/28562679/1407371).


# Cómo usar Git Worktree

Ver [artículo](https://morgan.cugerone.com/blog/how-to-use-git-worktree-and-in-a-clean-way/). Tengo resaltados en Omnivore.

Los git worktree son una forma de poder trabajar en distintas ramas de un mismo proyecto git. Esta es una alternativa a git stash.

En git stash guardo los cambios de manera temporal y luego me paso de rama. Ejemplo:

    git stash
    git checkout hotfix

Con un git worktree, se crea una nueva carpeta a partir de la rama que se especifique. Un ejemplo previo:

    ~/projects/devaspros/coshinotes
    
    # después de crear worktree
    
    ~/projects/devaspros/coshinotes
    ~/projects/devaspros/hotfix

donde la carpeta “hotfix” sigue siendo parte del mismo proyecto git que está en “coshinotes”.

Para crear un worktree nuevo, desde la carpeta donde regularmente se trabaj:

    git worktree add ../hotfix main

Donde:

- `../hotfix` es la carpeta a crear
    - nota que se crea por fuera de la carpeta actual
- `main` es la rama de la cual desprender la rama

Todo el trabajo que se haga en el worktree será independiente de la rama que esté en la carpeta “coshinotes”. Se puede hacer commit, push y eventualmente borrar la rama. Con todo listo podemos volver a la carpeta inicial y borrar el worktree.


    git worktree remove hotfix

El worktree crea la carpeta en base a su nombre. Así se pueden ver los worktress:

    git worktree list


## Organizar worktrees

En el artículo se explica una forma de tratar de trabajar con los worktrees para mantener una referencia a mismo nivel de las carpetas creadas.

La verdad eso me parece más embolatado así que no lo pienso revisar a fondo.

# Restaurar un archivo cambiado en un commit muy viejo

Pasó que se había modificado el archivo `db/schema.rb` en un commit muy viejo y no solo podía hacer git reset para restaurarlo. Entonces, ¿cómo podía devolverlo a ese estado anterior?

Se logra con el comando `git restore` (de git 2.23 en adelante).

```bash
git restore --source 1046ebfc4 db/schema.rb
```

[Este comentario](https://stackoverflow.com/a/57676529/1407371) menciona la forma con `git restore`.

donde a la bandera `--source` se le pasa el hash del commit al cual queremos devolver el archivo.

También se puede hacer lo mismo con `git checkout`:
```bash
git checkout c5f567~1 -- file1/to/restore file2/to/restore
```

Esta es la [forma antigua](https://stackoverflow.com/a/215731/1407371) que funciona en toda versión de git.

En nuestro caso sería así:
```bash
git checkout 1046ebfc4 -- db/schema.rb
```
