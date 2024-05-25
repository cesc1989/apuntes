# Manipulación de PDFs en Ruby
El estado de estas soluciones en el mundo Ruby es actualmente complicado.

La solución más estable de hace muchos años fue usar [WickedPDF](https://github.com/mileszs/wicked_pdf) que por debajo depende de [WKHTMLTOPDF](https://github.com/wkhtmltopdf/wkhtmltopdf) pero este último fue depreciado y archivado en 2022.

![](https://paper-attachments.dropboxusercontent.com/s_DFF467DAE3399DCE621D2BC67A15F10A9F2C6253A6858C8734EF7D2460374CBA_1692972508276_imagen.png)


En los proyectos de Luna que generan PDFs (Forms y Credentialing) estamos usando WickedPDF sin problemas pero el producto final es bastante pobre.

En este [pull request](https://github.com/mileszs/wicked_pdf/pull/733) menciona el OP que logró una configuración usando headless Chrome. Nota que el PR es de 2018 y en 2021 fue la última actualización.

# ¿Qué alternativas hay entonces?
## Usar otras librerías hechas en Ruby

Siempre que la necesidad sea de HTML a PDF se podrían usar alternativas ofrecidas mediante servicios o aquellas que usen Chrome headless mediante Puppeteer o similares.

## Crear un microservicio que use librerías de otros lenguajes

Esto nace porque ya ha salido [varias](https://wkhtmltopdf.org/status.html) [veces](https://github.com/zakird/wkhtmltopdf_binary_gem/issues/53#issuecomment-1003701135) la recomendación de usar [Weasyprint](https://github.com/Kozea/WeasyPrint/). Sin embargo, esta librería está escrita en Python.

## Servicios en Línea o APIs

[DocRaptor](https://docraptor.com/)
Usa Prince como backend.

> DocRaptor’s HTML-to-PDF API and supporting resources are designed to help developers quickly create a high-quality PDF.

[Prince](https://www.princexml.com/)
Es propietario y carisima las licencias.

> Convert HTML to PDF. Beautiful printing with simple CSS.
# Librerías Alternativas
## Manipulación Directa de PDF

[Prawn](https://github.com/prawnpdf/prawn)
Este no recibe HTML sino que genera el PDF mediante un DSL.

> Prawn is a pure Ruby PDF generation library that provides a lot of great functionality while trying to remain simple and reasonably performant.

[HexaPDF](https://github.com/gettalong/hexapdf)
Creación y manipulación de PDF.

> The main difference between HexaPDF and Prawn is that HexaPDF is a **full PDF library** whereas Prawn is a **library for generating content**.

[Origami](https://github.com/gdelugre/origami)
Framework para la manipulación de archivos PDF.

> Origami is a framework written in pure Ruby to manipulate PDF files.
> 
> It offers the possibility to parse the PDF contents, modify and save the PDF structure, as well as creating new documents.


## A partir de HTML

[Dhalang](https://github.com/NielsSteensma/Dhalang)
Es un wrapper de Puppeteer y facilita generar PDF desde HTML.

Mis primeras impresiones fueron: es bastante limitado y la configuración bastante rara. Hay que volverlo a probar.

[Weasyprint](https://github.com/simplybusiness/weasyprint)
Es un wrapper de la librería Weasyprint (hecha en Python).

> Create PDFs using plain old HTML+CSS. Uses [weasyprint](http://weasyprint.org/) on the back-end which renders HTML.

*Nota: la última actualización de repo fue en 2014. No tiene issues lo cual no es buena señal tampoco.*

El repo muestra que es un fork de PDFKit y la vez que probé esa librería no me funcionó tan bien. Hay que probar esta librería.

[Grover](https://github.com/Studiosity/grover)

> A Ruby gem to transform HTML into PDFs, PNGs or JPEGs using Google Puppeteer/Chromium

*Nota: Última actualización fue el 8 de Octubre de 2023.*


## Otros

[CombinePDF](https://github.com/boazsegev/combine_pdf)
Analiza y mezcla archivos PDF.

> CombinePDF is a nifty model, written in pure Ruby, to parse PDF files and combine (merge) them with other PDF files, watermark them or stamp them (all using the PDF file format and pure Ruby code).

Lo uso en Credentialing Form para mezclar los PDFs de formulario de registro, los términos y la firma del paciente en un único documento.

[Invoice Printer](https://github.com/strzibny/invoice_printer)
Solo para generar PDFs de facturas.

> **Super simple PDF invoicing.** InvoicePrinter is a server, command line program and pure Ruby library to generate PDF invoices in no time. You can use Ruby or JSON as the invoice representation to build the final PDF.


# Sobre Weasyprint

Es mencionada en varias partes aunque es una librería hecha en Python.

## Importante
- Es mantenida por una empresa: [CourtBoullion](https://github.com/Kozea/WeasyPrint/issues/1232).
- Sobre los [motivos](https://doc.courtbouillon.org/weasyprint/stable/going_further.html) para su creación.
- El wrapper que [existe en Ruby](https://github.com/simplybusiness/weasyprint) es un fork de PDFKit y está sin mantenimiento.

