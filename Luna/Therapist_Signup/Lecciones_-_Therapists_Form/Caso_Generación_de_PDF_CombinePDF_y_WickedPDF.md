# Caso Generación de PDF: CombinePDF y WickedPDF
Para ya salir del complique que es tener los textos de todos los agreements en un mismo archivo HTML, me di a la búsqueda de una solución para generar el archivo PDF donde se muestra que un usuario acepta los términos al registrarse en el Credentialing Form.

Por una sugerencia de Hemel y un listado de gemas para generar PDFs, probé la combinación de la gema CombinePDF mas lo que ya tenía de WickedPDF.

# La idea
- Tener los archivos agreements de cada estado en formato PDF en un bucket en S3.
- Descargarlos todos cada tanto tiempo a la carpeta `storage`.
- Cada que un usuario se registre:
    - Generar el archivo PDF de los campos del registro más la firma *SIN* el texto del agreement. En la carpeta `storage`.
    - Usando CombinePDF, cargar el archivo correspondiente al estado del usuario.
    - Guardar el nuevo archivo PDF resultado de la mezcla del archivo de los campos y firmas más el archivo del agreement según el estado. En la carpeta `storage`.
- El mailer que manda el adjunto del archivo del signup, cargará el archivo mezclado anteriormente y guardado en la carpeta `storage`.
- El trabajo en segundo plano que carga el archivo del signup a S3 debe buscar ahora el archivo generado en la carpeta `storage`.


# Notas

Encontré ciertas cosas sobre cómo funciona la generación de un PDF desde string en WickedPDF.


## Generar y Guardar

Resulta que solo hacer esto:

    pdf = WickedPdf.new.pdf_from_string(
      ActionController::Base.new.render_to_string(
        template: 'therapists/show.pdf.erb',
        layout: 'pdf.html',
        locals: { :@therapist => @therapist }
      )
    )

No crea un archivo PDF. En realidad sí lo hace pero en la carpeta temporal de sistema y ahí se pierden cuando el proceso se termina.

Para persistir el archivo y poder tomarlo luego hay que guardar el temporal donde necesitamos:

    pdf = WickedPdf.new.pdf_from_string(
      ActionController::Base.new.render_to_string(
        template: 'therapists/show.pdf.erb',
        layout: 'pdf.html',
        locals: { :@therapist => @therapist }
      )
    )
    
    File.open(Rails.root.join('storage', 'tmp_signed.pdf'), 'wb') do |file|
      file << pdf
    end


## Especificar ruta para guardar el archivo en CombinePDF

Para especificar la ruta en el método `save` solo hay que indicar la ruta relativa o absoluta:

    @final_pdf << CombinePDF.load(signed_pdf)
    @final_pdf << CombinePDF.load(agreement_pdf)
    
    @final_pdf.save('./storage/signed_agreement.pdf')


## Los Attachments de ActionMailer reciben un string

Cuando intenté pasar un archivo a attachments:

    attachments[signup_pdf_attachment] = File.open(Rails.root.join('storage', 'signed_agreement.pdf'), 'r')

Arrojaba error diciendo que `File` no tiene un método `length`.

Buscando encontré una [pista en WickedPDF](https://github.com/mileszs/wicked_pdf/#include-in-an-email-as-an-attachment) donde dice que attachments espera un string y luego en StackOver Flow donde [sugieren el método](https://stackoverflow.com/a/5145889/1407371) `[read](https://stackoverflow.com/a/5145889/1407371)`.


