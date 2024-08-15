# Lecciones Linux - Luna Patients

## Instalar software con pip

`pip` es una especie de gestor de paquetes del lenguaje Python.

Cuando intenté instalar `awslogs` había un error con respecto a intentar desinstalar unos paquetes previamente instalados en el sistema.

El error era similar a este:

    Installing collected packages: numpy
      Found existing installation: numpy 1.8.2
    Cannot uninstall 'numpy'. It is a distutils installed project and thus we cannot accurately determine which files belong to it which would lead to only a partial uninstall.

Se puede [solucionar usando una bandera](https://stackoverflow.com/questions/49932759/pip-10-and-apt-how-to-avoid-cannot-uninstall-x-errors-for-distutils-packages) para ignorar dichos paquetes:

    pip install awslogs --ignore-installed --user

Además, usar la bandera `--user` ayuda a que se instale a nivel de usuario y no de sistema(sería *root*).

## Instalación de `pip` y `pip3` en Linux Mint 19

Linux Mint 19 viene con Python3 por defecto y se debe instalar Python2.

Para instalar pip o pip3 en [Linux Mint procedemos de la siguiente forma](https://www.itzgeek.com/how-tos/linux/ubuntu-how-tos/how-to-install-pip-on-ubuntu-18-04-debian-9-linux-mint-19.html):

    # pip3
    sudo apt update
    sudo apt install python3-pip
    
    # Se verifica con
    pip3 --version
    
    # pip2
    sudo apt update
    sudo apt install python-pip
    
    # Se verifica con
    pip --version

## Condicionales en Bash y en Linux

Cuando estaba actualizando mis *dotfiles* porque al instalar `awslogs` y el `aws` CLI, aparecía un mensaje de advertencia sobre algunos binarios instalados con `pip` no disponibles en el `PATH`.

La solución era agregar la ruta a la variable `PATH` y lo hice con unos condicionales.

El aprendizaje está en que los corchetes(*square brackets*) de los condicionales en Bash son un alias del comando `test`.

Este comando ejecuta la condición dada. Para el caso de algunas instrucciones se hace con unas banderas. Si se ven las *man pages* de se pueden ver. Las que yo he visto más son:


- `-f` verifica que sea verdadera la condición y el archivo sea regular
    - Un [archivo regular](https://www.quora.com/What-is-a-Regular-file-in-Linux) es aquel que puede ser creado por el comando `touch`
- `-d` verifica que sea verdadera la condición y el archivo sea un directorio
- `-s` verifica que sea verdadera la condición y el tamaño del archivo sea mayor a cero

## Comando bash con barra inversa

Cuando se ve un comando como el siguiente:

    [ -s "$SOME_FILE" ] && \. "$SOME_FILE"

El *backslash* [significa que lo siguiente se debe interpretar tal cual](https://unix.stackexchange.com/questions/460533/what-does-backslash-dot-mean-as-a-command). En este caso, el carácter punto se interpretará como su comando normal que es *source*.

Sirve para casos donde puede que mediante un alias u otra forma se cambie la ejecución normal del punto(`.`)

## Configuración de AWS CLI en Linux

La configuración consta la creación de una carpeta `~./aws` y dos archivos dentro de esa carpeta `~/.aws/credentials` y `~./aws/config`.

[Se explica en la documentación de AWS](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html).

Contenido de `~./aws/credentials`:

    [default]
    aws_access_key_id=EXAMPLE
    aws_secret_access_key=EXAMPLE

Contenido de `~./aws/config`:

    [default]
    region=us-west-2
    output=json


## Error `dyld` de Homebrew con Node

Luego de una actualización de seguridad, reinicio del mac e instalación de otra versión de Ruby, Homebrew se toteó y dejó varios errores por ahí.

Este paso al intentar lanzar `npm start`:

    $ npm start
    dyld: Library not loaded: /usr/local/opt/icu4c/lib/libicui18n.64.dylib
      Referenced from: /usr/local/bin/node
      Reason: image not found
    Abort trap: 6

La solución se dio con `brew upgrade && brew cleanup`.

Visto en [Stack Overflow](https://stackoverflow.com/a/54873233/1407371).

