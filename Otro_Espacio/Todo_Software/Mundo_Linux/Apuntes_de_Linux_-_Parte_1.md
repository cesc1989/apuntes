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

Como explican en [Stack Overflow](https://stackoverflow.com/a/23585994/1407371)

> Bash functions work like shell commands and expect arguments to be passed to them in the same way one might pass an option to a shell command (...) _function arguments_ in Bash are treated as _positional parameters_ (`$1, $2..$9, ${10}, ${11}`),

Así se define:

```bash
function function_name {
   command...
}
```

Así se podría invocar con varios parémtros

```bash
function_name "$arg1" "$arg2"
```

**Enlaces**

- Passing parameters to a Bash function [Ver](https://stackoverflow.com/questions/6212219/passing-parameters-to-a-bash-function)
- Optional arguments:
	- [SO](https://stackoverflow.com/questions/9332802/how-to-write-a-bash-script-that-takes-optional-input-arguments)
	- [Shell parameter expansion](https://www.gnu.org/software/bash/manual/bashref.html#Shell-Parameter-Expansion)
	- [Example code](https://stackoverflow.com/a/33419280/1407371)

## Usar getopts para Named Arguments

Named arguments [Unix & Linux](https://unix.stackexchange.com/a/129401/47620)

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

> Nota que los parámetros son de una sola letra.

### Actualización: 11 de Octubre 2024

Usar `getopts` en funciones que se cargan en dotfiles puede no funcionar o hacer cosas raras.

Me tocó, con la ayuda gpt, hacer validaciones manuales y descartar `getopts`. Parecer ser que es mejor usar `getopts` en funciones que viven en archivos ejecutables.

## Named arguments sin getops

De esta otra forma se puede tener palabras en lugar de letras para los nombres de los argumentos.

Visto en [Stack Overflow](https://stackoverflow.com/a/14203146/1407371).

### Bash Space-Separated (e.g., `--option argument`)

Argumento separado por espacio en blanco.

```bash
#!/bin/bash

POSITIONAL_ARGS=()

while [[ $# -gt 0 ]]; do
  case $1 in
    -e|--extension)
      EXTENSION="$2"
      shift # past argument
      shift # past value
      ;;
    -s|--searchpath)
      SEARCHPATH="$2"
      shift # past argument
      shift # past value
      ;;
    --default)
      DEFAULT=YES
      shift # past argument
      ;;
    -*|--*)
      echo "Unknown option $1"
      exit 1
      ;;
    *)
      POSITIONAL_ARGS+=("$1") # save positional arg
      shift # past argument
      ;;
  esac
done

set -- "${POSITIONAL_ARGS[@]}" # restore positional parameters

echo "FILE EXTENSION  = ${EXTENSION}"
echo "SEARCH PATH     = ${SEARCHPATH}"
echo "DEFAULT         = ${DEFAULT}"
echo "Number files in SEARCH PATH with EXTENSION:" $(ls -1 "${SEARCHPATH}"/*."${EXTENSION}" | wc -l)

if [[ -n $1 ]]; then
    echo "Last line of file specified as non-opt/last argument:"
    tail -1 "$1"
fi
```

Se invoca de esta forma:
```bash
/tmp/demo-space-separated.sh -e conf -s /etc /etc/hosts
```

### Bash Equals-Separated (e.g., `--option=argument`)

Argumento separado por signo de igual.

```bash
#!/bin/bash

for i in "$@"; do
  case $i in
    -e=*|--extension=*)
      EXTENSION="${i#*=}"
      shift # past argument=value
      ;;
    -s=*|--searchpath=*)
      SEARCHPATH="${i#*=}"
      shift # past argument=value
      ;;
    --default)
      DEFAULT=YES
      shift # past argument with no value
      ;;
    -*|--*)
      echo "Unknown option $i"
      exit 1
      ;;
    *)
      ;;
  esac
done

echo "FILE EXTENSION  = ${EXTENSION}"
echo "SEARCH PATH     = ${SEARCHPATH}"
echo "DEFAULT         = ${DEFAULT}"
echo "Number files in SEARCH PATH with EXTENSION:" $(ls -1 "${SEARCHPATH}"/*."${EXTENSION}" | wc -l)

if [[ -n $1 ]]; then
    echo "Last line of file specified as non-opt/last argument:"
    tail -1 $1
fi
```

Y se invoca así:

```bash
/tmp/demo-equals-separated.sh -e=conf -s=/etc /etc/hosts
```


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

## Actualización 2024

En una unidad SSD esto no pasa. Parece ser problema meramente de HDD. Lo veo luego de ponerle el SSD al portátil Thinkpad y al pasar archivos a la USB o a los discos no se queda pegado en el 99%.

# ¿Diferencia entre Bash y sh?

Respuesta en [ask ubuntu](http://askubuntu.com/questions/141928/what-is-difference-between-bin-sh-and-bin-bash).

Bash y sh son shells de comandos. Bash es un predecesor de sh más moderno y con más características.

sh -> [Bourne shell](https://en.wikipedia.org/wiki/Bourne_shell). Corre en `/bin/sh`.

Bash -> [Bourne again shell](https://en.wikipedia.org/wiki/Bash_(Unix_shell)). Corre en `/bin/bash`.

Hoy en día, cuando se ve algo como `/bin/sh` suele ser un enlace simbólico que apunta a la shell de sistema.

# ¿Qué son las columnas del comando `ls -l`?

Respuestas en [Unix Exchange](https://unix.stackexchange.com/questions/103114/what-do-the-fields-in-ls-al-output-mean#103118)

Para el ejemplo de la pregunta en el enlace que da esta salida:
```bash
-rwxrw-r--    1    root   root 2048    Jan 13 07:11 afile.exe
```

`-rwxrw-r--` son permisos de archivo. Así:
```bash
-rwx # user
xr- # group
r-- # other
```

El primer carácter es un guión si el elemento es un archivo y es `d` si es un directorio.

`1` número de enlaces _hard_. **¿Qué son estos?**

`root` dueño del archivo.

`root` grupo (de sistema) del archivo.

`2048` peso del archivo en bytes.

`Jan 13 07:11` fecha de última modificación.

`afile.exe` nombre del archivo o directorio (si lo fuera)

# ¿Qué hace `/dev/null` en un script en bash?

Respuesta en [ask ubuntu](http://askubuntu.com/questions/514748/what-does-dev-null-mean-in-a-shell-script).

El OP pregunta por estos dos comandos:
```
cat /dev/null > /var/log/messages
cat /dev/null > /var/log/wtmp
```

En ese caso `/dev/null` es un [mecanismo que envía nada al archivo objetivo](https://askubuntu.com/a/514763/167553). Y aquí está la distinción entre enviar texto usando `>` o `>>`.

Donde al enviar texto con el operador `>` se indica que se eliminen los contenidos del archivo y luego se agregue el nuevo texto. En cambio con `>>` se agregan los nuevos contenidos al archivo sin borrar nada.

Por lo tanto:
```
cat /dev/null > /var/log/messages
```

Es una forma de limpiar el archivo. El archivo se limpia de esta forma porque si se borra con el comando `rm`, se pueden perder permisos.

En otra [respuesta](https://askubuntu.com/a/823708/167553) dicen que:
> /dev/null is like a black hole. Writes made to the /dev/null are discarded

Se usa para vaciar archivos. Otras formas de vaciar archivos son:
```
> /var/log/messages
 
truncate  -s  0  /var/log/messages
cp  /dev/null  /var/log/messages
```

# Vainas de cURL

## ¿Qué es el `bash -` al final de algunos comandos curl?

 Respuesta en [Ask Ubuntu](https://askubuntu.com/questions/703397/what-does-the-in-bash-mean).

Ejemplo:
```bash
curl --silent --location https://rpm.nodesource.com/setup | bash -
```

> the `-` is saying that there are **no more options**.

Si hubiera más palabras luego de `| bash` se tratarían como el nombre del archivo a ejecutar. En este caso se agrega el guión para indicarle a bash que no hay más argumentos.

Al final es lo mismo, en este caso, que no poner el guión.

## ¿Qué es la bandera `-o-` en comandos curl?

Respuesta en [Stack Overflow](https://stackoverflow.com/questions/40509660/unclear-curl-command-o).

Ejemplo, al instalar NVM:
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
```

Ahí pasan dos cosas. Una es la bandera `-o` es que para que curl de la salida a un archivo y la otra cosa es el guión `-`. Al usarse juntos `-o-` se está diciendo a curl que la salida de esa ejecución la mandé al `stdout` para que así la tome `| bash` como su entrada (`stdin`).

El comando igual funcionaría sin esto pero creo que a veces lo ponen para más claridad.

# Configuraciones para ssh

- config para ssh más brevinol: [Nerderati](http://nerderati.com/2011/03/17/simplify-your-life-with-an-ssh-config-file/)
- Configurar archivo `~/.ssh/config` para que cargue las llaves configuradas en el mismo: [Super User](https://superuser.com/questions/325662/how-to-make-ssh-agent-automatically-add-the-key-on-demand#1114257) - [Ask Ubuntu](https://askubuntu.com/a/853578/167553)


# ¿Cuál es el comando de Sublime desde Terminal?

Respuesta en [Ask Ubuntu](https://askubuntu.com/questions/524812/run-sublime-text-3-and-check-version#524815).

El comando es `subl`. Ejemplos:

```bash
$ which subl

/usr/local/bin/subl
```

```bash
$ subl --version

Sublime Text Build 4169
```

# Comando `command`

El comando `command` http://pubs.opengroup.org/onlinepubs/9699919799/utilities/command.html
> The command utility shall cause the shell to treat the arguments as a simple command

He visto en tutoriales de instalación u otros usar este comando. Más que nada lo vi para identificar el path del ejecutable de algún programa.
```bash
$ command -v subl

/usr/local/bin/subl
```

¿Qué es lo que hace? Esto dice los docs:
> Write a string to standard output that indicates the pathname or command that will be used by the shell

# ¿Qué es `$@` en scrips de Bash?

En [Stack Overflow](https://stackoverflow.com/questions/9994295/what-does-mean-in-a-shell-script)

Explicado en [esta respuesta](https://stackoverflow.com/a/9995322/1407371):
> `$@` is nearly the same as `$*`, both meaning "all command line arguments". They are often used to (...) pass all arguments to another program (thus forming a wrapper around that other program).

Ejemplo. Este script `./someScript.sh foo bar` then `$@` will be equal to `foo bar`. Dentro del script se puede referenciar usando `$@`.

```bash
# super_script.sh

umbrella_corp_options "$@"
```

A la función/comando `umbrella_corp_options` se le pasa la lista de parámetros, cada uno encerrado en comillas.

# Mover archivos ocultos con el comando mv

En [Ask Ubuntu](https://askubuntu.com/questions/259383/how-can-i-get-mv-or-the-wildcard-to-move-hidden-files)

Esta [respuesta](https://askubuntu.com/a/259386/167553) dice que se logra así:
```bash
shopt -s dotglob
mv /tmp/home/rcook/* /home/rcook/
```

Otra forma, más sencilla al parecer, sería:
```bash
mv a/{.*,*} b/
```

¿Qué hacen las llaves? Esa es la sintaxis de _brace expansion_. Así lo explica la [misma respuesta](https://askubuntu.com/a/1332761/167553). Esto es una forma de generar un rango de valores.

Sin la _brace expansion_ el comando sería así:
```bash
mv a/.* b/
mv a/* b/
```

# Enlaces que faltan por volver apuntes

## Bash Scripting

- [Parameter Expansion detailed](https://unix.stackexchange.com/a/122848/47620) - [More on Stack Overflow](https://stackoverflow.com/questions/2013547/assigning-default-values-to-shell-variables-with-a-single-command-in-bash)

## Cosas de Ubuntu

- Algunos campos de texto pierden el foco al presionar la tecla CTRL: [la respuesta es desactivar mostrar el cursor al presionar CTRL del sistema](https://support.mozilla.org/es/questions/1191486) - [Como hacerlo en Ask Ubuntu](https://askubuntu.com/questions/230102/how-do-i-turn-off-show-mouse-when-i-press-ctrl)

## Cosas de Linux en General

- Cron Tasks linux: [crontab](http://www.thegeekstuff.com/2009/06/15-practical-crontab-examples) - [Cron job](https://askubuntu.com/questions/2368/how-do-i-set-up-a-cron-job)
- nginx: `emerg could not build the server_names_hash, you should increase either server_names_hash_max_size: 256 or server_names_hash_bucket_size: 64` [ver solución](https://serverfault.com/questions/419847/nginx-setting-server-names-hash-max-size-and-server-names-hash-bucket-size)
- Al truncar un archivo el disco puede no tener espacio porque el proceso aún está usando el archivo: [Superuser](https://superuser.com/a/738698/372807)
- unix wildcard [double asterisk](http://stackoverflow.com/questions/3529997/unix-wildcard-selectors-asterisks)
- [service restart vs service reload](https://askubuntu.com/questions/105200/what-is-the-difference-between-service-restart-and-service-reload)
- `truncate` para limpiar archivos: [Linux DIE](https://linux.die.net/man/1/truncate) - [Unix & Linux](https://unix.stackexchange.com/a/88810/47620)
- `diff` and `cmp` commands: [Use](https://stackoverflow.com/questions/3611846/bash-using-the-result-of-a-diff-in-a-if-statement) diff in if statement
- USB no se montaban automáticamente: [Borrar](https://superuser.com/a/788454/372807) `[/etc/mtab.fuselock](https://superuser.com/a/788454/372807)` [y reiniciar](https://superuser.com/a/788454/372807) - [Linux Format Foro](https://www.linuxformat.com/forums/viewtopic.php?p=109844)
- Listar servicios de sistema: [service --status-all](https://stackoverflow.com/questions/18721149/check-if-a-particular-service-is-running-on-ubuntu)
- Atajo de linux mint para capturar pantalla con área seleccionada y mandarla al portapapeles no funca bien: [la solución es presionar rápidamente SHIFT y luego PRINT mientras se presiona CTRL](https://github.com/linuxmint/Cinnamon/issues/5634#issuecomment-244530211)
- Configurar shortcuts en Linux Mint [Ask Ubuntu](https://askubuntu.com/questions/170163/how-do-i-set-a-shortcut-to-screenshot-a-selected-area)
- Para arreglar problema de llaves GPG invalidas al actualizar software, prueba añadiendo de nuevo la llave indicada en el proceso de instalación(a lo mejor ya se venció) - [Ver caso Yarn](https://github.com/yarnpkg/yarn/issues/4453#issuecomment-329463752)
- Sobre [bluetooth en Linux Mint](https://maslinux.es/como-configurar-bluetooth-en-gnulinux/). [rfkill](https://linux.die.net/man/1/rfkill) me sirvió para saber que el bluetooth estaba bloqueado
