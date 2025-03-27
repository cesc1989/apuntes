# Instalando gemas de AWS de manera independiente

Tenemos instalado la gema `aws-sdk-s3` y necesitaba instalar `aws-sdk-sesv2`. No sabía qué versión de esta última instalar. ¿Cómo encontrarlo?

Probé instalar la más reciente pero daba conflicto porque `aws-sdk-s3` instala `aws-sdk-core` [versión 1.80.3](https://rubygems.org/gems/aws-sdk-core/versions/3.180.3). Necesitaba encontrar la versión de la gema de SES que dependiera de una versión de core menor de 1.80.3.

Así que fui a Ruby Gems y visité la página de las diferentes tags de la gema ses. En esa página hay un apartado donde muestra las dependencias y ahí podía ver la versión de core que necesitaba y así di con [la versión 1.39.0](https://rubygems.org/gems/aws-sdk-sesv2/versions/1.39.0) que depende de core 3.177.0 y así pude instalar la gema por fín.