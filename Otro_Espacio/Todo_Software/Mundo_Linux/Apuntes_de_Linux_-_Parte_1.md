# Apuntes de Linux - Parte 1

# Redireccionar `STDERR` y `STDOUT` a archivo

En [Stack Overflow](http://stackoverflow.com/questions/637827/redirect-stderr-and-stdout-in-a-bash-script).

**Sobre Standard Output, Standard Input y Standard Error**

- Explicar qué es output, input y error en Linux
- Explicar STDOUT, STDIN Y STDERR
- Sobre la redirecciń de STDOUT, STDIN Y STDERR con `>`, `>>`, `&`, `1>`, `2>`, `&>`, `/dev/null` y `2>&1`
> 1 = STDOUT, 2 = STDERR, & = STDOUT y STDERR
- Acá está una explicación para todo esto: [Thoughtbot](https://robots.thoughtbot.com/input-output-redirection-in-the-shell)

# Agregar repositorio APT sin confirmación del usuario

Agregar repositorio apt no-interactivo:

    sudo apt-add-repository -y ppa:[REPOSITORY-NAME]
    sudo apt-add-repository -y ppa:ondrej/php
    sudo apt-add-repository -y ppa:ansible/ansible


# Argumentos en funciones Bash

Como explican en [Stack Overflow](https://stackoverflow.com/a/23585994/1407371):

> In bash function, arguments are referenced by position rather than name. Positions are in the range of `$1..$n`

Así se define:
```bash
function function_name {
   command...
}
```

Así se invoca
```bash
function_name "$arg1" "$arg2"
```

## Enlaces

- Passing parameters to a Bash function [Ver](https://stackoverflow.com/questions/6212219/passing-parameters-to-a-bash-function)
-  Named arguments:
	- [Unix & Linux](https://unix.stackexchange.com/a/129401/47620)
	- [getopts](https://unix.stackexchange.com/a/129401/47620)
- Named arguments [without and with getops](https://stackoverflow.com/a/14203146/1407371)
- Optional arguments:
	- [SO](https://stackoverflow.com/questions/9332802/how-to-write-a-bash-script-that-takes-optional-input-arguments)
	- [Shell parameter expansion](https://www.gnu.org/software/bash/manual/bashref.html#Shell-Parameter-Expansion)
	- [Example code](https://stackoverflow.com/a/33419280/1407371)


## Ejemplo de cómo usar getopts

```bash
function namedWithGetOps () {
  while getopts ":p:s:" opt; do
    case $opt in
      p) first="$OPTARG"
      ;;
      s) second="$OPTARG"
      ;;
      \?) echo "Invalid option -$OPTARG" >&2
      ;;
    esac
  done

  printf "Argument first is %s\n" "$first"
  printf "Argument second is %s\n" "$second"
}

namedWithGetOps -p primero -s segundo
```

Sintaxis:

1. `getopts OPTSTRING VARNAME [ARGS...]`
2. Indicate var receives an argument following its name by a colon: `s:`
3. A colon at the beginning of the options string silents error reporting: `:p`


# Limpieza en Linux Mint

Desinstalar paquetes instalados con `dpkg`

Para buscar un paquete desinstalable con esta utilidad:

    dpkg-query -l atom

Para desinstalarlo:

    sudo dpkg -P atom


- Limpiar `/var/cache`: [Ask Ubuntu](https://askubuntu.com/a/367619/167553)
- Cómo desinstalar JDownloader 2: [Ask Ubuntu](https://askubuntu.com/questions/393181/how-to-uninstall-jdownloader-2-beta)


# Correr programa en modo dettached

Correr programas *dettached* significa que el proceso no esté pegado a la terminal y que el STDOUT no vaya a la pantalla

- [Superuser 1](https://superuser.com/questions/177218/how-to-start-gui-linux-programs-from-the-command-line-but-separate-from-the-com)
- [Superuser](https://superuser.com/questions/178587/how-do-i-detach-a-process-from-terminal-entirely)

Ejemplo:
`Postman &> /dev/null &`

# Problema de transferencia a USB que se queda en el 99% por mucho tiempo

Este problema fue lo que me hizo dañar mi disco portable de 1TB.

Al transferir archivos de Linux Mint, mediante USB, si la unidad receptora es lenta o el puerto no es muy rápido, la transferencia se quedará en *99%* por un tiempo desconocido.

En este [issue](https://github.com/linuxmint/nemo/issues/964) hay más información al respecto y [acá se explica](https://unix.stackexchange.com/questions/180818/gnome-nautilus-copy-files-to-usb-stops-at-100-or-near/181236#181236) por qué ocurre.

**Resumen**
Cuando se transfieran archivos grandes, tener paciencia cuando se "detenga" en el 99%.

# Enlaces que faltan por volver apuntes

## Bash Scripting

- /dev/null [ask ubuntu](http://askubuntu.com/questions/514748/what-does-dev-null-mean-in-a-shell-script)
- bash vs sh [ask ubuntu](http://askubuntu.com/questions/141928/what-is-difference-between-bin-sh-and-bin-bash)
- Move hidden files with mv command: [ask ubuntu](https://askubuntu.com/questions/259383/how-can-i-get-mv-or-the-wildcard-to-move-hidden-files)
- Format or interpolate date: [SO - 1](https://stackoverflow.com/questions/23655580/in-bash-how-do-i-interpolate-in-a-string) - [SO - 2](https://stackoverflow.com/questions/1401482/yyyy-mm-dd-format-date-in-shell-script). Also `$(date +%F %T)`
- Assign command output to variable in bash: [Unix & Linux](https://unix.stackexchange.com/questions/16024/how-can-i-assign-the-output-of-a-command-to-a-shell-variable) - [Command substitution](http://tldp.org/LDP/abs/html/commandsub.html)
- [Parameter Expansion detailed](https://unix.stackexchange.com/a/122848/47620) - [More on Stack Overflow](https://stackoverflow.com/questions/2013547/assigning-default-values-to-shell-variables-with-a-single-command-in-bash)
- Continue command: [SO](https://stackoverflow.com/questions/7316107/bash-continuation-lines) - [NixCraft](https://www.cyberciti.biz/faq/howto-ask-bash-that-line-command-script-continues-next-line/)
- [Explanation of](https://unix.stackexchange.com/questions/103114/what-do-the-fields-in-ls-al-output-mean#103118) `[ls -l](https://unix.stackexchange.com/questions/103114/what-do-the-fields-in-ls-al-output-mean#103118)` [command](https://unix.stackexchange.com/questions/103114/what-do-the-fields-in-ls-al-output-mean#103118)
- Comando de Sublime Text en línea de comandos: [Ask Ubuntu](https://askubuntu.com/questions/524812/run-sublime-text-3-and-check-version#524815)
- explicación a comandos, sobre todo `curl` donde al final le pasan el resultado a bash pero bash lo sigue un guión: `curl ... | bash -`: [Ask Ubuntu](https://askubuntu.com/questions/703397/what-does-the-in-bash-mean)
- explicación de bandera `-o-` del comando *curl*: [Stack Overflow](https://stackoverflow.com/questions/40509660/unclear-curl-command-o)
- El comando `command`: [command](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/command.html) *"The command utility shall cause the shell to treat the arguments as a simple command"*
- qué es `$@` en scrips de Bash: [SO](https://stackoverflow.com/questions/9994295/what-does-mean-in-a-shell-script)

## Cosas de Ubuntu

- `[nodejs](https://chrislea.com/2014/07/09/joining-forces-nodesource/)` [PPA repo is not anymore Chris Lea](https://chrislea.com/2014/07/09/joining-forces-nodesource/)
- Install latest nodejs version in Ubuntu 14.04: [SO](http://stackoverflow.com/questions/34974535/install-latest-nodejs-version-in-ubuntu-14-04)
- Cómo desinstalar Ruby que viene por defecto en el sistema: [installation.co](http://installion.co.uk/ubuntu/xenial/main/r/ruby/uninstall/index.html) - [SO](https://stackoverflow.com/questions/3957730/how-can-i-uninstall-ruby-on-ubuntu)
- Algunos campos de texto pierden el foco al presionar la tecla CTRL: [la respuesta es desactivar mostrar el cursor al presionar CTRL del sistema](https://support.mozilla.org/es/questions/1191486) - [Como hacerlo en Ask Ubuntu](https://askubuntu.com/questions/230102/how-do-i-turn-off-show-mouse-when-i-press-ctrl)


## Cosas de Linux en General

- Cron Tasks linux: [crontab](http://www.thegeekstuff.com/2009/06/15-practical-crontab-examples) - [Cron job](https://askubuntu.com/questions/2368/how-do-i-set-up-a-cron-job)
- nginx: emerg could not build the server_names_hash, you should increase either server_names_hash_max_size: 256 or server_names_hash_bucket_size: 64 [ver solución]](https://serverfault.com/questions/419847/nginx-setting-server-names-hash-max-size-and-server-names-hash-bucket-size)
- Al truncar un archivo el disco puede no tener espacio porque el proceso aún está usando el archivo: [Superuser](https://superuser.com/a/738698/372807)
- config para ssh más brevinol: [Nerderati](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/)
- unix wildcard [double asterisk](http://stackoverflow.com/questions/3529997/unix-wildcard-selectors-asterisks)
- [service restart vs service reload](https://askubuntu.com/questions/105200/what-is-the-difference-between-service-restart-and-service-reload)
- `truncate` para limpiar archivos: [Linux DIE](https://linux.die.net/man/1/truncate) - [Unix & Linux](https://unix.stackexchange.com/a/88810/47620)
- `diff` and `cmp` commands: [Use](https://stackoverflow.com/questions/3611846/bash-using-the-result-of-a-diff-in-a-if-statement) diff in if statement
- USB no se montaban automáticamente: [Borrar](https://superuser.com/a/788454/372807) `[/etc/mtab.fuselock](https://superuser.com/a/788454/372807)` [y reiniciar](https://superuser.com/a/788454/372807) - [Linux Format Foro](https://www.linuxformat.com/forums/viewtopic.php?p=109844)
- [Explain Shell](https://explainshell.com/)
- Aliases para comandos: [How to geek](https://www.howtogeek.com/73768/how-to-use-aliases-to-customize-ubuntu-commands/)
- Listar servicios de sistema: [service --status-all](https://stackoverflow.com/questions/18721149/check-if-a-particular-service-is-running-on-ubuntu)
- Agrandar espacio de disco de máquina Vagrant: [Ask Ubuntu](https://askubuntu.com/questions/317338/how-can-i-increase-disk-size-on-a-vagrant-vm)
- Atajo de linux mint para capturar pantalla con área seleccionada y mandarla al portapapeles no funca bien: [la solución es presionar rápidamente SHIFT y luego PRINT mientras se presiona CTRL](https://github.com/linuxmint/Cinnamon/issues/5634#issuecomment-244530211)
- Configurar shortcuts en Linux Mint [Ask Ubuntu](https://askubuntu.com/questions/170163/how-do-i-set-a-shortcut-to-screenshot-a-selected-area)
- Para arreglar problema de llaves GPG invalidas al actualizar software, prueba añadiendo de nuevo la llave indicada en el proceso de instalación(a lo mejor ya se venció) - [Ver caso Yarn](https://github.com/yarnpkg/yarn/issues/4453#issuecomment-329463752)
- Configurar archivo `~/.ssh/config` para que cargue las llaves configuradas en el mismo: [Super User](https://superuser.com/questions/325662/how-to-make-ssh-agent-automatically-add-the-key-on-demand#1114257) - [Ask Ubuntu](https://askubuntu.com/a/853578/167553)
- Sobre [bluetooth en Linux Mint](https://maslinux.es/como-configurar-bluetooth-en-gnulinux/). [rfkill](https://linux.die.net/man/1/rfkill) me sirvió para saber que el bluetooth estaba bloqueado
