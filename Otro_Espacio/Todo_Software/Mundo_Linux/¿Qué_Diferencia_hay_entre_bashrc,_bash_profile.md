# ¿Qué Diferencia hay entre .bashrc y .bash_profile?

¿Qué diferencia hay entre estos archivos de inicialización de sistema?

**Enlaces**

- [Stack Overflow](https://stackoverflow.com/questions/415403/whats-the-difference-between-bashrc-bash-profile-and-environment)
- [Server Fault](https://serverfault.com/questions/261802/what-are-the-functional-differences-between-profile-bash-profile-and-bashrc)
- [Super User](https://superuser.com/questions/789448/choosing-between-bashrc-profile-bash-profile-etc)

La principal diferencia es entre los que sirven para shells interactivos o shells de login. Los archivos `.bashrc` y `.bash_profile` son específicos de Bash shell.

La man page de Bash nos dice esto:
```
FILES
   /bin/bash
        The bash executable
   /etc/profile
        The systemwide initialization file, executed for login shells
  ~/.bash_profile
        The personal initialization file, **executed for login shells**
   ~/.bashrc
        The individual **per-interactive-shell** startup file
   ~/.bash_logout
        The individual login shell cleanup file, executed when a login shell exits
   ~/.inputrc
        Individual readline initialization file
```

## Otros Detalles

Según [esta respuesta](https://stackoverflow.com/a/415444/1407371) en Stack Overflow, Bash complica la cosa porque el archivo `.bashrc` es leído por **solamente por shell que sea interactiva y no-login**. Ejemplo: Terminal.

El archivo `~/.profile` puede ser usado por cualquier otra shell que no tenga archivo propio de configuración.

# ¿Qué es un shell de login?

Según [ChatGPT](https://chatgpt.com/share/022cc4c1-8f60-4f63-bc8e-3d4761a7c82a), es un shell que se inicia cuando el usuario se autentica en un sistema operativo Unix.

La página man de Bash nos dice que:
> A login shell is one whose first character of argument zero is a -, or one started with the --login option.

Un login shell ejecutará varios archivos de configuración. Aquí es donde vemos a `.bash_profile`.

![[login.shell.files.png]]

Esto dice la página man:
> When bash is invoked as an interactive login shell, or **as a non-interactive shell with the --login option**, it first reads and executes commands from the file `/etc/profile`, if that file exists.  After reading that file, it looks for `~/.bash_profile`, `~/.bash_login`, and `~/.profile`, in that order, and reads and executes commands from the first one that exists and is readable.

Aquí tenemos que la diferencia entre `.bash_profile` y `.bashrc`. El primero es leído por shells de login O interactivas.

En este vídeo de Full Stack Zach, vemos como ejecuta una shell de login y se escribe a pantalla los archivos que carga y en el orden que fueron cargados.

![[login.shell.example.png]]

1. bash_profile
2. profile
3. bashrc

Nota como también está cargando `.bashrc`.

# ¿Qué es una shell interactiva?

Una shell interactiva es la que permite la interacción directa con el usuario. En este modo la shell muestra un prompt y permite que el usuario escriba comandos a los cuales la shell devuelve salidas y errores.

Una shell interactiva ejecuta scripts específicos para este fin. En el caso de Bash es el archivo `.bashrc`.

> Es por esto que para [[Usando Zellij]] me tocó configurar la cargada de los aliases en el `.bashrc` ya que Zellij es una shell interactiva.

Es una shell que se abre desde un entorno gráfico. Por ejemplo, abrir Terminal en Macos o en Linux.

La man page nos dice que:
> An interactive shell is one started without non-option arguments and without the `-c` option whose standard input and error are both connected to terminals, or one started with the `-i` option.

# ¿Por qué no se carga lo de `.profile` cuando se abre una nueva terminal?

- [Why ~/.bash_profile is not getting sourced when opening a terminal?](https://askubuntu.com/questions/121073/why-bash-profile-is-not-getting-sourced-when-opening-a-terminal)
- [Bash not loading '.profile' in new session on Linux](https://superuser.com/questions/176404/bash-not-loading-profile-in-new-session-on-linux)

Los archivos `~/.bash_profile` o `~/.profile` se cargan solo en shells de login. Ejemplo de shell logins:

- Conectar mediante SSH
- sudo -i
- su -

Abrir una nueva ventana de Terminal es una shell en modo interactivo (no login) por eso no se carga ese archivo. En cambio cargará la configuración que esté en `~/.bashrc`.


**Meta**

Etiquetas: #linux #bashrc