# Configuración de Marketplace

> [!Warning]
> Tener cuidado con la versión de Python. Es clave que los comandos se corran para la versión especificada en el README, .python-version o al final de Pipfile.
>
> Versión al 25/09/25: 3.12.9

Hay que instalar pyenv y pipenv:

- Pyenv → https://github.com/pyenv/pyenv/#usage
    - switch between multiple versions of Python
    - como rbenv
- Pipenv → https://pipenv.pypa.io/en/latest/installation/

> Pipenv automatically creates and manages a virtualenv for your projects, as well as adds/removes packages from your `Pipfile` as you install/uninstall packages. It also generates a project `Pipfile.lock`, which is used to produce deterministic builds.

Para pipenv hay que agregar el binario al PATH
```bash
export PATH="$HOME/.local/bin:${PATH}"
```

Instalar Python con pyenv
```bash
pyenv install 3.12.9
```

Salida de ejemplo:
```
python-build: use openssl@3 from homebrew
python-build: use readline from homebrew
Downloading Python-3.11.7.tar.xz...
-> https://www.python.org/ftp/python/3.11.7/Python-3.11.7.tar.xz
Installing Python-3.11.7...
python-build: use readline from homebrew
python-build: use ncurses from homebrew
python-build: use zlib from xcode sdk
Installed Python-3.11.7 to /Users/francisco/.pyenv/versions/3.11.7
```

Para listar las versiones de python instaladas:
```bash
pyenv versions
```

Ejemplo de salida:
```
system
2.7.18
* 3.9.7 (set by /Users/francisco/projects/luna-project/marketplace/.python-version)
3.10.11
```

## Pyenv

Para que se cargue la versión requerido de Python se pueden usar alguno de estos comandos:

- pyenv shell
- pyenv local

### Detalle de pyenv shell

Cuando intenté con `pyenv shell` pasó esto:
```bash
pyenv shell 3.12.9
pyenv: shell integration not enabled. Run `pyenv init' for instructions.
```

Así que probé con `pyenv local`:
```bash
pyenv local 3.12.9
```

El cual funcionó pero creo un archivo `.python-version`
```bash
$ cat .python-version 
3.12.9
```

## Pipenv

**Importante**: Creo que fue importante este paso antes de instalar

> [!INFO]
> Así es como se crea el virtualenv

```bash
pipenv --python $(pyenv which python3.12)
```

Salida de ejemplo:
```
Loading .env environment variables...
Warning: the environment variable LANG is not set!
We recommend setting this in ~/.profile (or equivalent) for proper expected behavior.
Creating a virtualenv for this project...
Pipfile: /Users/francisco/projects/luna-project/marketplace/Pipfile
Using /Users/francisco/.pyenv/versions/3.11.7/bin/python3.11 (3.11.7) to create virtualenv...
⠙ Creating virtual environment...created virtual environment CPython3.11.7.final.0-64 in 901ms
  creator CPython3Posix(dest=/Users/francisco/.local/share/virtualenvs/marketplace-UlKaqkiD, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/Users/francisco/Library/Application Support/virtualenv)
    added seed packages: pip==23.2.1, setuptools==68.2.0, wheel==0.41.2
  activators BashActivator,CShellActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator

✔ Successfully created virtual environment!
Virtualenv location: /Users/francisco/.local/share/virtualenvs/marketplace-UlKaqkiD
```

Al modificar variables de entorno u otra cosa es clave **correr los comando en este orden antes de entrar en el ambiente virtual**:
```bash
pipenv --python $(pyenv which python3.12)

Loading .env environment variables...
```

luego:
```bash
pipenv shell
```

Salida de ejemplo:
```
Loading .env environment variables...
Loading .env environment variables...
Launching subshell in virtual environment...

. /Users/francisco/.local/share/virtualenvs/marketplace-UlKaqkiD/bin/activate
```

### Usando Pipenv para instalar dependencias

Luego de ejecutar `pipenv install` para instalar los paquetes del Pipfile, dice esto:
```bash
pipenv install --dev
```

Salida de ejemplo:
```
Loading .env environment variables...
Installing dependencies from Pipfile...
Installing dependencies from Pipfile...
To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.
```

#### Detalle de pipenv shell

Cuando ejecuto `pipenv shell` me da este error de bash:
```bash
pipenv shell

Launching subshell in virtual environment...

The default interactive shell is now zsh.
To update your account to use zsh, please run `chsh -s /bin/zsh`.
For more details, please visit https://support.apple.com/kb/HT208050.
bash: prompt_git: command not found
```

Y parece entrar en una subshell pero cada que presiono algo vuelve ese error
```bash
. /Users/francisco/.local/share/virtualenvs/marketplace-UlKaqkiD/bin/activate
bash: prompt_git: command not found
(marketplace) 

 
bash: prompt_git: command not found
(marketplace)
```

**¿Qué está pasando?**
Es un tema con la forma en que bash le hereda configuraciones a las subshells.

**Solución**
Solución vista en [este issue](https://github.com/pypa/pipenv/issues/3712).

La solución está dada por incluir, en mi caso, incluir el archivo que cambia el prompt de la consola en los archivos `.bash_profile` y `.bashrc`:

En mis dotfiles se hace lo primero:
```bash
# Load all current dotfiles
for DOTFILE in $HOME/projects/dotfiles/system/.{alias,prompt}; do
	[ -f "$DOTFILE" ] && source $DOTFILE
done
```

Para el archivo `.bashrc` toca hacerlo manual:
```bash
# Include personal dotfiles prompt_git function into subshells.
[ -f "$HOME/projects/dotfiles/system/.prompt" ] && source $HOME/projects/dotfiles/system/.prompt
```

Cuando recargo el shell y pruebo de nuevo:
```bash
pipenv shell

Launching subshell in virtual environment...

The default interactive shell is now zsh.
To update your account to use zsh, please run `chsh -s /bin/zsh`.
For more details, please visit https://support.apple.com/kb/HT208050.
 . /Users/francisco/.local/share/virtualenvs/marketplace-UlKaqkiD/bin/activate
```

### Otros comandos de pipenv

```bash
# Show the path of the virtual environment (directory)
pipenv --venv

# Remove the local virtual environment with the whole installed packages
pipenv --rm
```

## pip3

Para instalar una versión específico de una librería en pip3 toca así:
```bash
pip3 install flask==2.3.3
```

Ejemplo:
```bash
$ pip3 install flask==2.3.3
	 ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 96.1/96.1 kB 2.1 MB/s eta 0:00:00
Installing collected packages: flask
Successfully installed flask-2.3.3
```

# Lanzar un servidor flask

Primero necesitamos generar una BD backup en alpha para usarla:
```bash
alpha_marketplace_generate_dump
```

Después se usa la respuesta que arroja en el canal eng-infra para obtener las credenciales:
```bash
luna rds get-dev-db-creds -p alpha -d dev-francisco-alpha-marketplace-05-27t21-17
```

En esta parte el primer comando es seguido por el del README pero parece que basta solo con el `pipenv shell` para cargar las variables de entorno en el ambiente virtual:
```bash
pipenv --python $(pyenv which python3.12)

pipenv shell
```

## Flask Shell (como Rails console)

Hay que movernos a la carpeta app para lanzar la shell de Flask:
```python
cd app/
FLASK_APP=marketplace/application.py flask shell
```

Ya dentro de la shell podemos lanzar diferentes comandos de librerias
```python
from marketplace.visit_plan import jobs
jobs.write_visit_plans_to_data_lake()

from marketplace.visit_plan import jobs;jobs.write_visit_plans_to_data_lake()
```

## Flask Server

Para lanzar el servidor web ponemos las variables de entorno en el archivo `.env`
```bash
REDIS_URL=redis://localhost:6379
FLASK_ENV=development
FLASK_APP=marketplace/application.py

# Si se usa macos, esta debe ir
export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES
```

Así se lanza el web server:
```bash
flask run
```

y así para lanzar un worker:
```bash
rq worker -c marketplace.settings_rq_worker_default
```

# Ejecutar Flask local

Teniendo claro que el comando pipenv instala todos los paquetes y que había que correr el comando descrito en el README, al fin pude hacer de manera exitosa:
```bash
cd app/
FLASK_APP=marketplace/application.py flask shell

INFO:root:marketplace.visit_plan.listeners does not exist; skipping
Python 3.9.7 (default, Oct  9 2023, 10:16:51) 
[Clang 14.0.0 (clang-1400.0.29.202)] on darwin
App: marketplace.application
Instance: /Users/francisco/projects/luna-project/marketplace/app/instance
>>> 
```

Aunque dice otra versión de Python…

Para ejecutar el servidor web:
```bash
REDIS_URL=redis://localhost:6379 FLASK_ENV=development FLASK_APP=marketplace/application.py flask run

INFO:root:marketplace.visit_plan.listeners does not exist; skipping
 * Serving Flask app 'marketplace/application.py'
 * Debug mode: on
INFO:werkzeug:WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:5000
INFO:werkzeug:Press CTRL+C to quit
INFO:werkzeug: * Restarting with stat
```

# ENV Vars

Estas son la variables de entorno que he visto son necesarias. Algunas no se mencionan.

En el archivo `app/marketplace/config/base.py` están todas las que se podrían necesitar.

Son necesarias para poder acceder a la base de datos que funciona como dump pero en una instancia de RDS.
```bash
RDS_DB_NAME=luna
RDS_USERNAME=lunacareadmin
RDS_PASSWORD=postgres
RDS_HOSTNAME=localhost
RDS_PORT=5432
```

Otras más
```bash
FLASK_APP=marketplace/application.py

REDIS_JOBS_URL=redis://redis:6379/1
REDIS_URL=redis://redis:6379/0
RQ_DASHBOARD_PASSWORD=clavesegura

DATA_LAKE_BUCKET=somelakebucket
```

# Error al correr rq

De vez en mes salía este error:
```
objc[21706]: +[__NSCFConstantString initialize] may have been in progress in another thread when fork() was called.
objc[21706]: +[__NSCFConstantString initialize] may have been in progress in another thread when fork() was called.
```

La solución es exportar una variable:
```
export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES
```

# Las Migraciones

> [!Important]
> Resulta que no debo correr migraciones sino usar un dump

Actualmente no se necesita correr migraciones. Hay es que configurar una base de datos con copia de datos en alpha y accederla mediante un conexión string.

## Error de timeout

Si por alguna razón empiezo a tener problemas de Timeout aunque las credenciales estén bien, hay que ejecutar este comando del Luna CLI

```
luna rds whitelist-ip-for-dev-db --profile alpha
```

# Error al instalar pillow-heif

Esta librería [pillow-heif](https://pillow-heif.readthedocs.io/en/latest/installation.html) me da este error al hacer `pipenv install --dev`:
```
  error: subprocess-exited-with-error

  × Building wheel for pillow-heif (pyproject.toml) did not run successfully.
  │ exit code: 1
  ╰─> [81 lines of output]
  
    note: This error originates from a subprocess, and is likely not a problem with pip.
  ERROR: Failed building wheel for pillow-heif
ERROR: Failed to build installable wheels for some pyproject.toml based projects (pillow-heif)
ERROR: Couldn't install package: {}
 Package installation failed...
```

Probé instalando por aparte:
```
python3 -m pip install --upgrade pillow-heif
Collecting pillow-heif
  Downloading pillow_heif-1.1.1-cp312-cp312-macosx_11_0_arm64.whl.metadata (9.6 kB)
Requirement already satisfied: pillow>=11.1.0 in /Users/francisco/.local/share/virtualenvs/marketplace-UlKaqkiD/lib/python3.12/site-packages (from pillow-heif) (11.3.0)
Downloading pillow_heif-1.1.1-cp312-cp312-macosx_11_0_arm64.whl (3.5 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 3.5/3.5 MB 5.9 MB/s  0:00:00
Installing collected packages: pillow-heif
Successfully installed pillow-heif-1.1.1
```

Pero eso tampoco sirvió al final.

## Solución de instalación de pillow-heif

Según chatgpt, ya la librería está instalada. El problema es que `pip install --dev` está tratando de crear una nueva "wheel" desde cero y ahí es donde falla. Eso lo pude comprobar con todo esto.

Importar directo desde Flask
```
python -c "import pillow_heif; print(pillow_heif.__version__)"
```

Ese comando devolvió `1.1.1`. Lo que indica que sí está instalado.
