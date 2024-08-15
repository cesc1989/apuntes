# Lecciones Herramientas - Luna Patients

## Interceptar Peticiones Salientes desde Aplicación Móvil

Resulta que para poder descubrir qué estaba causando el error donde un formulario, al guardarse su borrador, resultaba teniendo muchísimos *medicamentos, aggravating_activitie*s e incluso *answers*.

Una de las formas es usando **Android Studio** pero es muy tedioso instalar y configurar tal IDE. Las opciones con Android Studio son el [Network Profiler](https://developer.android.com/studio/profile/network-profiler) y para Android en general en [este artículo se describen varias](https://proandroiddev.com/various-methods-to-debug-http-traffic-in-the-android-application-8685b9183418) pero se tienen que integrar con la aplicación.

Otra forma está dada usando una herramienta llamada **Charles**.

[Instalé y usé Charles](https://www.charlesproxy.com/) en el Macbook Pro pero las peticiones que salen desde la aplicación móvil de Luna son HTTPS y las respuestas están encriptadas. Parece que es posible verlas pero hay que configurar un certificado SSL en el *.apk* de la aplicación.

La otra alternativa era [usando el proxy nativo de Postman](https://learning.getpostman.com/docs/postman/sending_api_requests/capturing_http_requests/) pero en la aplicación nativa no hay forma de capturar peticiones que van por HTTPS ya que generalmente tienen algo que se llama HSTS y Postman no tiene soporte para eso.

Por lo tanto Postman solo queda para interceptar peticiones HTTP.


> **Note:** for the Postman native apps, request captures over HTTPS will not work if the website has [HSTS](https://es.wikipedia.org/wiki/HTTP_Strict_Transport_Security) enabled. Most websites have this check in place.

También estaba la extensión *postman interceptor* pero solo captura lo que ocurre en el navegador y no desde la aplicación móvil.

## Configuración y Andada de APK en Android Studio

En este sitio se detalla el proceso para [instalar y configurar Android Studio en macOS](https://hdorgeval.gitbooks.io/setup-your-mac-to-develop-nativescript-apps/content/install-android-studio-on-mac-os-x.html) además de cómo [configurar el Java JDK](https://hdorgeval.gitbooks.io/setup-your-mac-to-develop-nativescript-apps/content/install-jdk-on-mac-os-x.html)(aunque es solo descargar e instalar).

También me salió un error al cargar el apk:


    Error: select Android SDK

Ya tenía instalado el SDK necesario y [había creado los](https://hdorgeval.gitbooks.io/setup-your-mac-to-develop-nativescript-apps/content/setup-android-emulator-on-mac-os-x.html) [*Virtual Devices*](https://hdorgeval.gitbooks.io/setup-your-mac-to-develop-nativescript-apps/content/setup-android-emulator-on-mac-os-x.html) y salía ese error. La solución fue la siguiente([vista en Stack Overflow](https://stackoverflow.com/questions/27133211/error-select-android-sdk)):


    From your project:
    1. File->Project Structure
    2. In Android SDK location select the SDK directory

