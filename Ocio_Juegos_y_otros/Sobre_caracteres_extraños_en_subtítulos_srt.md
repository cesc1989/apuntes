# Sobre caracteres extraños en subtítulos .srt
Estaba tratando de ver “No Time To Die” en el TV (Hyundai) y los signos de pregunta o tildes salían cómo caracteres ASCII.

El texto en los subtítulos estaba bien, el detalle era la codificación del archivo.

Resulta que la codificación debe ser o UTF-8 o ANSI. Si el archivo ya está en UTF-8, hay que probar pasarlo a ANSI y viceversa.

Para ver la codificación del archivo se puede usar el comando `file` en Mac o Linux:

    $ file -bI ~/Downloads/No.Time.To.Die.2021.1080p.BluRay.x264.AAC5.1-\[YTS.MX\].srt 
    text/plain; charset=utf-8

En este caso, el archivo ya era UTF-8 y estaba molestando. Debía pasarlo a ANSI.

Cuando intentaba pasarlo usando `iconv`, el archivo quedaba vacío:

    $ iconv -t MS-ANSI ~/Downloads/No.Time.To.Die.2021.1080p.BluRay.x264.AAC5.1-\[YTS.MX\].srt > ~/Downloads/No.Time.To.Die.2021.1080p.BluRay.x264.AAC5.1-\[YTS.MX\].srt

Usando Sublime Text también se puede hacer la conversión yendo al submenú “Archivo → Guardar con Codificación” sin embargo, la opción “ANSI” no aparece (o no sé cual es). Para ello tuve que usar Windows, abrir y guardar el archivo usando Bloc de Notas.

Finalmente, al pasarlo a ANSI, aparecieron bien los subs en el TV.

