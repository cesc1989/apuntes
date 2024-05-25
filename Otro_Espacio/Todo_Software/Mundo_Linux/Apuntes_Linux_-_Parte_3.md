# Apuntes Linux - Parte 3

# Cómo buscar texto en un grupo de archivos

¿Cómo se puede encontrar un texto en una carpeta de proyecto? Esto sería en lugar de usar la búsqueda global de Sublime, por ejemplo.

Recursos:

- [How can I find all files containing a specific text (string) on Linux?](https://stackoverflow.com/questions/16956810/how-can-i-find-all-files-containing-a-specific-text-string-on-linux)
## Solución: usando grep

Así explican en esta respuesta

    grep -Rnw '/path/to/somewhere/' -e 'pattern'


- `-r` or `-R` is recursive ; use `-R` to search entirely
- `-n` is line number, and
- `-w` stands for match the whole word.
- `-l` (lower-case L) can be added to just give the file name of matching files.
- `-e` is the pattern used during the search

Along with these, `--exclude`, `--include`, `--exclude-dir` flags could be used for efficient searching:

This will only search through those files which have .c or .h extensions:

    grep --include=\*.{c,h} -rnw '/path/to/somewhere/' -e "pattern"

This will exclude searching all the files ending with .o extension:

    grep --exclude=\*.o -rnw '/path/to/somewhere/' -e "pattern"

For directories it's possible to exclude one or more directories using the `--exclude-dir` parameter. For example, this will exclude the dirs `dir1/`, `dir2/` and all of them matching `*.dst/`:

    grep --exclude-dir={dir1,dir2,*.dst} -rnw '/path/to/search/' -e "pattern"


## Solución mega bacana: instala The Silver Searcher

The Silver Searcher https://github.com/ggreer/the_silver_searcher


> A code searching tool similar to `ack`, with a focus on speed.

Instalación:

    # mac
    brew install the_silver_searcher
    
    # Ubuntu
    apt-get install silversearcher-ag

Uso:

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

Y cómo el texto no le hace justicia, una captura:

![](https://paper-attachments.dropboxusercontent.com/s_47EC6AA6B80BBC219CDA2D94AF7673323A6B07A2CA20F1784E7CD8ABD5C41B94_1698421265077_imagen.png)


Está buenísimo.

# ¿Cómo listar los contenidos de un archivo plano de abajo hacía arriba?

En linux para listar los contenidos de archivos se puede usar el comando `cat`.

    $ cat archivo.txt
    uno
    dos
    tres

Resulta que si necesito listar los contenidos de un archivo, en orden al revés, existe una herramienta que se llama tac:

    $ tac archivo.txt
    tres
    dos
    uno

También se pueda lograr lo mismo con `tail -r` pero no parece no hay soporte para todos los sistemas operativos.

Fuente: [Stack Overflow](https://stackoverflow.com/questions/742466/how-can-i-reverse-the-order-of-lines-in-a-file).

