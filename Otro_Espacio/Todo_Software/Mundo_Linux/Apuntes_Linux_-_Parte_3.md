# Apuntes Linux - Parte 3

# Cómo buscar texto en un grupo de archivos

¿Cómo se puede encontrar un texto en una carpeta de proyecto? Esto sería en lugar de usar la búsqueda global de Sublime, por ejemplo.

Recursos:

- [How can I find all files containing a specific text (string) on Linux?](https://stackoverflow.com/questions/16956810/how-can-i-find-all-files-containing-a-specific-text-string-on-linux)

## Solución: usando grep

Así explican en esta respuesta
```
grep -Rnw '/path/to/somewhere/' -e 'pattern'
```

- `-r` or `-R` is recursive ; use `-R` to search entirely
- `-n` is line number, and
- `-w` stands for match the whole word.
- `-l` (lower-case L) can be added to just give the file name of matching files.
- `-e` is the pattern used during the search

Along with these, `--exclude`, `--include`, `--exclude-dir` flags could be used for efficient searching:

This will only search through those files which have .c or .h extensions:
```
grep --include=\*.{c,h} -rnw '/path/to/somewhere/' -e "pattern"
```

This will exclude searching all the files ending with .o extension:
```
grep --exclude=\*.o -rnw '/path/to/somewhere/' -e "pattern"
```

For directories it's possible to exclude one or more directories using the `--exclude-dir` parameter. For example, this will exclude the dirs `dir1/`, `dir2/` and all of them matching `*.dst/`:
```
grep --exclude-dir={dir1,dir2,*.dst} -rnw '/path/to/search/' -e "pattern"
```

## Solución mega bacana: instala The Silver Searcher

The Silver Searcher https://github.com/ggreer/the_silver_searcher

> A code searching tool similar to `ack`, with a focus on speed.

Instalación:
```
# mac
brew install the_silver_searcher

# Ubuntu
apt-get install silversearcher-ag
```

Uso:
```bash
ag 'form_submitted'

app/models/therapist.rb
5:  scope :form_submitted, -> { where(form_completed: true).where.not(form_completed_at: nil) }
56:  def form_submitted?
73:    return if form_submitted?

app/controllers/api/base_api_controller.rb
18:      return unless therapist.form_submitted?
47:      return if therapist.form_submitted?

app/views/therapists/index.html.erb
19:          <%= therapist.form_submitted? %>

app/services/hubspot_resync_service.rb
29:    if @therapist.form_submitted? && !form_status_manually_edited?

app/services/hubspot_contact_service.rb
57:    if @therapist.form_submitted?

spec/requests/api/mini/therapists_spec.rb
28:      let(:therapist) { FactoryBot.create(:therapist_with_main_form_completed, :mini_form_submitted) }

spec/factories/therapists.rb
126:    trait :mini_form_submitted do

lib/tasks/003_sync_therapists_info_to_hubspot.rake
46:            Rails.logger.info("Did #{therapist.email} submitted form? #{therapist.form_submitted?}")
48:            service.new(therapist: therapist).submit_form if therapist.form_submitted?

lib/tasks/005_create_pdf_packets.rake
7:      therapists = Therapist.form_submitted.where(registered_from: state)
9:      therapists = Therapist.form_submitted
```

Y cómo el texto no le hace justicia, una captura:

![[silver.searcher.png]]

Está buenísimo.

# ¿Cómo listar los contenidos de un archivo plano de abajo hacía arriba?

En linux para listar los contenidos de archivos se puede usar el comando `cat`.
```bash
cat archivo.txt
uno
dos
tres
```

Resulta que si necesito listar los contenidos de un archivo, en orden al revés, existe una herramienta que se llama tac:
```bash
tac archivo.txt
tres
dos
uno
```

También se pueda lograr lo mismo con `tail -r` pero no parece no hay soporte para todos los sistemas operativos.

Fuente: [Stack Overflow](https://stackoverflow.com/questions/742466/how-can-i-reverse-the-order-of-lines-in-a-file).

# Herramientas y utilidades que compartió DHH mediante Omakub

Bacano que se haya puesto a probar Linux. Debido a que se puso en esas supe de la existencia de Zellij y vaya que está bacano. Estas son algunas de las herramientas o software incluidos en [Omakub](https://omakub.org/):

- [Pinta](https://www.pinta-project.com/): edición de imágenes
- [Flameshot](https://flameshot.org/): para captura de pantalla
	- Se ve mucho mejor que los por defecto en macos y Linux Mint
- [eza](https://github.com/eza-community/eza?tab=readme-ov-file): un `ls` más poderoso y moderno
- [ripgrep](https://github.com/BurntSushi/ripgrep): reemplazo de `grep`
- [zoxide](https://github.com/ajeetdsouza/zoxide?tab=readme-ov-file): alternativa moderna de `cd`

Hay que revisarlas y probarlas.

# Cómo copiar carpeta desde terminal

El comando `cp` solo no puede copiar la carpeta contenedora. Hay que usar `cp -r`. Ejemplo:
```bash
cp -r ../patient-forms-backend/spec/forms ./spec/forms/patient_self_report/
```

# Revertir la actualización de Núcleo de Linux

En la mañana del 2 de Julio tenía actualizaciones en Linux Mint que incluían una de núcleo. Hubo varios problemas para completar porque al parecer no se podía descargar algo. Cancelé un par de veces y volví a intentar pero nunca terminaba la descarga.

Reinicié y los problemas aparecieron. La pantalla tomó una resolución mucho menor y no se reconocía la tarjeta de wifi.

Le fui a preguntar a ChatGPT y aquí están los pasos de lo que hay que hacer.

## Encontrar el núcleo actual

Para saber si es problema del núcleo necesitaba primero saber cual está en uso. Con este comando se puede saber:
```bash
uname -r
5.15.0-143-generic
```

## Entrar al modo avanzado de Grub

Me pedía que ingresara al Grub pero no lograba mostrarlo. Debido a que tengo Linux Mint instalado como único SO en el PC este no se mostraba.

Para poder mostrarlo tuve que editar el archivo `/etc/default/grub` así:
```
# GRUB_TIMEOUT_STYLE=hidden
GRUB_TIMEOUT=5
```

Guardé los cambios y luego ejecuté `sudo update-grub` para que se aplicaran. Reinicié y pude ver el menú Grub.

## Opciones Avanzadas de Linux Mint

Me aparecían varias opciones de núcleos. Como la dañada era la versión 143-generic tenía que bajar a una versión antes. Me salía esto:

```
- 142-generic
- 142-generic (recovery mode)
```

Aquí elegí la primera que es donde todo estaba funcionando correctamente.

## Borrar el núcleo problemático

Me tocó borrar varias cosa para poder borrar el 143.

Con
```bash
dpkg -l | grep 143
```

Listé todos los paquetes relacionados a este núcleo.

Intenté borrar con este comando:
```bash
sudo dpkg --purge linux-image-5.15.0-143-generic linux-modules-5.15.0-143-generic
```

Pero había dependencias que necesitaba borrar primero.

Tuve que borrarlas así:
```bash
sudo dpkg --purge linux-generic

sudo dpkg --purge linux-image-generic
```

Después si pude borrar los paquetes mencionados antes. Al ejecutar `dpkg -l | grep 143` no había resultados.

# Bash strict mode con `set` y `pipefail`

Quería mejorar los scripts de los despliegues de mis apps Ruby on Rails de tal forma que los fallos detuvieran el proceso enseguida. Pasaba que había fallos pero el script seguía. Era exitoso pero al revisar los logs se veían los errores.

Para eso DeepSeek recomendó usar esta combinación:
```bash
set -euxo pipefail
```

Al final, por cuestiones de chruby me tocó solo quedarme con `set -eo pipefail`. En todo caso quiero explicar qué hace cada opción.

## Hacer que el script falle con la instrucción `set`

Tomado de este gist: https://gist.github.com/akrasic/380bda362e0420be08709152c91ca1f9

Usar `set` hace que el script falle. Ocurre cuando se usa con ciertas banderas que al detectar un error hacen que el script falle de manera inmediata y haciendo bulla. Esto es positivo justo por las razones que comentaba. Es mejor que el script explote a que complete con errores.

Esto:
```bash
set -euxo pipefail
```

Es lo mismo que:
```bash
set -e
set -u
set -x
set -o pipefail
```

Veamos.

- La instrucción `set -e` le dice a bash que termine de manera inmediata si cualquier comando devuelve un estado que no sea cero (0)
	- Este es muy util en scripts pero no en una shell.
- La instrucción `set -x` activa un modo de la shell que hace que todos los comandos que se ejecuten sean imprimidos en la terminal.
	- Claro uso para ayudar en el debuggeo.
- La instrucción `set -u` afecta las variables. Cuando se define hace que el script falle si se intenta usar variables no definidas.
	- Esta fue la que no pude usar por una variable de chruby que nunca pude definir.
- La instrucción `set -o pipefail` previene que errores de un pipeline de comandos sean ocultados. Si cualquier comando en la cadena de instrucciones falla, el código de estado será el usado como la salida final del script.
	- La clave de esto es que si alguno de los diferentes comandos devuelve un código que no sea cero, el script falla y se de por terminado.

> [!Info]
> Con más detalle: http://redsymbol.net/articles/unofficial-bash-strict-mode/