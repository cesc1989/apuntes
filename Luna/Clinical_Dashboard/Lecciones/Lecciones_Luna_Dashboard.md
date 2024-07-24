# Lecciones Luna Dashboard

## ¿Cómo ordenar un hash con números como valores?

Si tenemos este *hash*:

    h = {
      "Neck"=>55,
      "Shoulder/Arm"=>101,
      "Upper Back"=>10,
      "Lower Back"=>163,
      "Hip"=>167,
      "Pelvis"=>7,
      "Knee"=>201,
      "Ankle/Foot"=>41,
      "Other"=>228
    }

¿Cómo hacemos para ordenarlo de tal forma que el valor y llave mayor sean los primero y bajando?

Así (visto en [Stack Overflow](https://stackoverflow.com/a/2540473/1407371)):

    h.sort_by { |_, v| v }.reverse.to_h
    => {"Other"=>228, "Knee"=>201, "Hip"=>167, "Lower Back"=>163, "Shoulder/Arm"=>101, "Neck"=>55, "Ankle/Foot"=>41, "Upper Back"=>10, "Pelvis"=>7} 

Otra forma un poco más rustica:

    h.invert.sort.reverse.to_h.invert

`Hash#invert` lo que hace es cambiar las llaves y las vuelve valores, y los valores del hash los convierte en las llaves del mismo.

## ¿Cómo enviar cabeceras HTTP cuando se hacen pruebas con RSpec?

La forma es:

    post "#{base_api_url}/links", params: {}, headers: { 'Authorization' => 'TOKEN' }

Visto en [Stack Overflow](https://stackoverflow.com/a/50978369/1407371).

## ¿Cómo usar la definición de una constante en una clase hija?

Para las gráficas de *body parts* hice una estructura jerárquica para poder reutilizar mejor el código.

Quedó más o menos así:

- `BaseBodyPartsConverter` → clase padre
    - `HipChartConverter`
    - `KneeChartConverter`
    - `LowerBackChartConverter`
    - `NeckChartConverter`
    - `ShoulderChartConverter`

Las clases hijas definen una constante de nombre `MCID` que tiene diferentes valores según la clase.

En la clase base, para poder usar dicha constante tocó hacer referencia de esta forma:


    @mcid_cases += 1 if operation >= self.class::MCID

Visto en [Stack Overflow](https://stackoverflow.com/questions/3174563/how-to-use-an-overridden-constant-in-an-inheritanced-class).


## Error de Query String demasiado largo

El error es:

    QUERY_STRING is longer than the (1024 * 10) allowed length (was 10787)
    
    HTTP parse error, malformed request (/v1/body_parts): #<Puma::HttpParserError: HTTP element QUERY_STRING is longer than the (1024 * 10) allowed length (was 12884)>

Ocurre cuando se hace una petición a cualquier endpoint para cualquier gráfica y se mandan demasiados valores en el parámetro `doctors[]=`.

En los navegadores, el *query string* tiene un límite por motivos de implementación y seguridad.

[Ver issue sobre Puma](https://github.com/puma/puma/issues/757#issuecomment-130153810).
For reference:

- MSIE - max total url length (inc path, etc) - 2,083 characters.
- Chrome - stops displaying after 64k characters, but can serve more than 100k characters.
- Firefox - same as Chrome.
- Safari - will not truncate and can handle up to 100k.
- Apache - ~4,000 characters, after which Apache produces a "413 Entity Too Large" error.
- IIS - 16,384 characters, but is configurable.
- Perl HTTP::Daemon - 8,000 bytes. After which a 413 is issued.

 
Una solución sería buscar la forma de tener una configuración personalizada para el proyecto pero no serviría a largo plazo porque muy seguramente podría pasar de nuevo.

También averigüe sobre enviar un cuerpo de petición en una petición GET pero [es desaconsejado](https://stackoverflow.com/questions/978061/http-get-with-request-body).

La mejor opción es recibir ese listado por petición POST.

## Error de libx11-dev

Se presentó este error al intentar crear un contenedor:

    ERROR: libx11-dev-1.6.10-r0: trying to overwrite usr/include/X11/extensions/XKBgeom.h owned by xorgproto-2018.4-r0.

Lo único que aparecía en internet era unos mensajes de unos intercambios de correo. Al final, se solucionó subiendo la versión de Ruby de 2.5.3 a 2.5.8


## Error de cURL para hacer petición de `/v1/links`

Cuando intentaba ejecutar este comando generado por Postman

    $ curl --location --request POST 'https://physicians.luna.farzoo.click/v1/links/?partners[]=IRG%20Physical%20Therapy' \
    → --header 'Authorization: Token eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1ODc0OTA2OTUsImdyb3VwX25hbWUiOiJDYWxpZm9ybmlhIFBhY2lmaWMgT3J0aG9wZWRpY3MifQ.cQKX0lOHhI9ZYmvUPawrBa-9c79_HSUiZhkJ_8Q_kE0'

Obtenía el error

    curl: (3) [globbing] bad range specification in column 57

Por alguna razón, el token con puntos le está dando problemas. La solución es usar la bandera `--globoff`. Así:


    $ curl --location --globoff  --request POST 'https://physicians.luna.farzoo.click/v1/links/?partners[]=IRG%20Physical%20Therapy' --header 'Authorization: Token eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1ODc0OTA2OTUsImdyb3VwX25hbWUiOiJDYWxpZm9ybmlhIFBhY2lmaWMgT3J0aG9wZWRpY3MifQ.cQKX0lOHhI9ZYmvUPawrBa-9c79_HSUiZhkJ_8Q_kE0'
    
    {"data":[{"IRG Physical Therapy":"/dashboard/eyJhbGciOiJIUzI1NiJ9.eyJleHAiOjE2MDQ2NzY0ODQsInBhcnRuZXJfbmFtZSI6IklSRyBQaHlzaWNhbCBUaGVyYXB5In0.WPwMv7JTYkUJuD2_X2IeJluqEa-Pd6Wv2pFsQ5tWr5Y"}]}

Visto en [Stack Overflow](https://stackoverflow.com/questions/43676800/solr-query-via-curl-globbing-bad-range-in-column).


## Comparando dos archivos con diff y git diff

Visto en [Superuser](https://superuser.com/questions/578359/how-can-i-enable-colored-output-for-osx-diff).

Existe el comando `diff` para comparar archivos:

    diff ~/Downloads/new_charts_qa.txt ~/Downloads/new_charts.saumil.txt


    diff ~/Downloads/new_charts_qa.txt ~/Downloads/new_charts.saumil.txt
    9c9
    < thumbs up: 207
    ---
    > thumbs up: 203
    15,17c15,17
    < Promoters: 108 (90%)
    < Passives: 9 (8%)
    < Detractors: 3 (3%)
    ---
    > Promoters: 104 (86%)
    > Passives: 13
    > Detractors: 4 (3%)
    19c19
    < NPS: 87%
    ---
    > NPS: 83%
    27,29c27,29
    < thumbs up: 540
    < thumbs down: 2
    < Percentage: 100
    ---
    > thumbs up: 582
    > thumbs down: 3
    > Percentage: 99
    33,35c33,35
    < Promoters: 113 (82%)
    < Passives: 18 (13%)
    < Detractors: 6 (4%)
    ---
    > Promoters: 117 (85%)
    > Passives: 16
    > Detractors: 4 (3%)
    37c37

pero la salida, en macos, es sin color. Para un ojo acostumbrado a Git, que mejor que una salida con color. Para fortuna, se puede usar `git diff`.


    git diff ~/Downloads/new_charts_qa.txt ~/Downloads/new_charts.saumil.txt


    diff --git a/Users/fquintero/Downloads/new_charts_qa.txt b/Users/fquintero/Downloads/new_charts.saumil.txt
    index 1b2b0cb..34e722b 100644
    --- a/Users/fquintero/Downloads/new_charts_qa.txt
    +++ b/Users/fquintero/Downloads/new_charts.saumil.txt
    @@ -6,17 +6,17 @@
     
     PATIENT SATISFACTION
     
    -thumbs up: 207
    +thumbs up: 203
     thumbs down: 1
     Percentage: 100
     
     NPS
     
    -Promoters: 108 (90%)
    -Passives: 9 (8%)
    -Detractors: 3 (3%)
    +Promoters: 104 (86%)
    +Passives: 13
    +Detractors: 4 (3%)
     
    -NPS: 87%
    +NPS: 83%
     
     ####
     ## Dr. Alexander Sah
    @@ -24,17 +24,17 @@ NPS: 87%
     
     PATIENT SATISFACTION
     
    -thumbs up: 540
    -thumbs down: 2
    -Percentage: 100
    +thumbs up: 582
    +thumbs down: 3
    +Percentage: 99

