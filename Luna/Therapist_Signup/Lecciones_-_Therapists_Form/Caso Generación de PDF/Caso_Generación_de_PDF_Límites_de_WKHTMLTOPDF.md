# Caso Generación de PDF: Límites de WKHTMLTOPDF
Como siempre, la situación está dada por la necesidad de generar un archivo PDF. En el caso del formulario de registro de terapeutas de Luna hay que generar uno para estos una vez que se registren.

Lo normal, en el mundo de Ruby(on Rails) es usar la gema [WickedPDF](https://github.com/mileszs/wicked_pdf/) la cual funciona como un *wrapper* de WKHTMLTOPDF. En términos generales, esta gema basta y sobra para generar PDFs sencillos sin muchos estilos ni estructuras complejas(ej. las tablas se rompen entre páginas), sin embargo, **sí hay una limitante**. Su [limitante](https://github.com/mileszs/wicked_pdf/issues/750#issuecomment-403616774) está dada por su dependencia.

Al querer *reutilizar* los estilos que ya Lobo hizo para la aplicación frontend y debido a que la app frontend usa características muy nuevas y avanzadas de CSS:


- Flexbox
- CSS Grid
- [CSS Variables(custom properties)](https://developer.mozilla.org/en-US/docs/Web/CSS/Using_CSS_custom_properties)

se complica al momento de generar el PDF ya que el software [WKHTMLTOPDF](https://wkhtmltopdf.org/) no soporta, aún, tales características:


- [Github/wkhtmltopdf: flexbox layout doesn't work](https://github.com/wkhtmltopdf/wkhtmltopdf/issues/1522)
- [Github/wkhtmltopdf: CSS grid layout - supported?](https://github.com/wkhtmltopdf/wkhtmltopdf/issues/3661)
- [Github/wkhtmltopdf: CSS: Grid Template Display support](https://github.com/wkhtmltopdf/wkhtmltopdf/issues/3964)
- [Github/wkhtmltopdf: CSS Variables Support](https://github.com/wkhtmltopdf/wkhtmltopdf/issues/3253)

Todo se debe a que la herramienta(al menos en su versión 0.12.4) usa una versión de muy vieja del motor de renderizado Qt/WebKit. Tal versión es un equivalente de usar **Google Chrome versión 13**(la versión actual es la 75).

## ¿Hay alternativas viables?

En primera instancia, la alternativa más viable para no cambiar la herramienta(que es la más actualizada en el ecosistema Ruby) es no usar tales características avanzadas y optar por las características de CSS más estandarizadas y presentes en navegadores.

Sin embargo, como tales funcionalidades son ideales para poder hacer un mejor trabajo e incluso facilitar hacerlo rápido y hacer cambios,  se debe pensar en cambiar a otra herramienta que haga renderizado del [navegador en modo](https://developers.google.com/web/updates/2017/04/headless-chrome) [*headless*](https://developers.google.com/web/updates/2017/04/headless-chrome) y que pueda generar un archivo PDF.


> Ejemplo de lo anterior es [Google Puppeteer](https://github.com/GoogleChrome/puppeteer). Documentación de [generar un archivo PDF de una página usando Puppeteer](https://pptr.dev/#?product=Puppeteer&version=v1.18.1&show=api-pagepdfoptions).

[Prawn](https://github.com/prawnpdf/prawn) no es una alternativa ya que está pensada para generar PDFs usando su DSL y no a partir de un HTML. También está [PDFKit](https://github.com/pdfkit/pdfkit) pero usa por debajo WKHTMLTOPDF, lo cual nos deja en la misma situación que WickedPDF.

Otras opciones son [HexaPDF](https://github.com/gettalong/hexapdf), [CombinePDF](https://github.com/boazsegev/combine_pdf) e [Invoice Printer](https://github.com/strzibny/invoice_printer) pero son para casos de uso donde no se genera PDF a partir de HTML.

## Usando Puppeteer

Algunos artículos y tutos:

- [PDF generation in Ruby using Google's Puppeteer, on Heroku](https://www.forsbergplustwo.com/blogs/news/pdf-generation-with-chrome-headless-in-ruby-using-puppeteer-on-heroku), 2017

Hay un par de gemas que parecen ser *wrappers* de Puppeteer en Ruby:

- [Dhalang](https://github.com/NielsSteensma/Dhalang)
- [PDFgen](https://github.com/romaimperator/pdfgen)
## En otros Lenguajes de Programación

En Elixir hay un modulo que dejó de usar WKHTMLTOPDF para usar Puppeteer y lo comparten en [este](https://github.com/richeterre/jumubase/issues/11) [*issue*](https://github.com/richeterre/jumubase/issues/11) y en [este artículo](https://medium.com/coletiv-stories/puppeteer-vs-wkhtmltopdf-and-why-i-created-a-new-module-9466eb1db7d1) en Medium.

