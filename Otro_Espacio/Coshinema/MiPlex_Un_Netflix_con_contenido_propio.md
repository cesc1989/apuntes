# MiPlex: Un Netflix con contenido propio

Con un precio que parece módico, usando Cloudflare Streaming puedo tener la parte más complicada de este proyecto: almacenamiento, servidores y transmisión.

Probé subir dos vídeos: uno X y una película y transmite normal la película.

## ¿Qué faltaría?

- UI para acceder a la lista de películas
- UI para ver película
    - Manejo de subs
- Sesión de usuarios
- Script para carga de archivos a Cloudflare Streaming

## ¿Y los subtítulos?

Bueno, todo bien cuando la película está en audio latino pero cuando necesita subtítulos hay que hacer otra cosa.

[Usando la API](https://developers.cloudflare.com/stream/uploading-videos/adding-captions), se agregan los subtítulos. Hay que convertirlos de `.srt` a `.webvtt`. Herramienta [sugerida](https://subtitletools.com/convert-to-vtt-online) por Cloudflare para convertir.

```bash
curl -X GET "https://api.cloudflare.com/client/v4/user/tokens/verify" \
	 -H "Authorization: Bearer TOKEN" \
	 -H "Content-Type:application/json"
```

Código para subir subtitulo:
```bash
curl -X PUT \
      -H 'Authorization: Bearer TOKEN' \
      -F 'file=/Users/fquintero/Downloads/Wall.Street.REMASTERED.1987.1080p.BrRip.x264.YIFY.vtt' \
      https://api.cloudflare.com/client/v4/accounts/396a0a05347c10523d2e2d4fed985b4a/stream/77058720aa74f7ad82481a0496f48014/captions/es
```

Bueno, la petición no servía y en este [hilo del foro](https://community.cloudflare.com/t/decoding-error-using-the-cloudflare-stream-api-to-upload-captions/197566/13) de Cloudflare parecen no encontrar solución.

Resulta que los *captions* se pueden subir desde el mismo tablero de Stream.

![[01.cloudflare.stream.subs.png]]

Se escoge el idioma y aparece una caja de selección de archivo.

Referencia del [API de Stream](https://api.cloudflare.com/#stream-subtitles/captions-upload-a-caption/subtitle). Sobre WebVTT ([estándar y validador](https://github.com/silviapfeiffer/webvtt)).

**Marzo 11**: subí subtítulo de película Wall Street luego de convertirlo a VTT y funciona!

# Resumen

Luego de probar subir películas y sus respectivos subtítulos, el proyecto se vuelve viable pero con algunos detalles.

Para tener todo el contenido en Cloudflare me toca:

- Me toca programar un *script que lea las películas desde el disco duro y las mande a Cloudflare*
- De cada película que *tenga subtítulos, me toca convertirlas a WebVTT y luego cargarlas a Cloudflare por el API o por la UI*

Debo encontrar la forma de calcular la cantidad de minutos de las películas que subiré para saber el precio mensual que deberé pagar.

Con lo anterior definido, lo siguiente será crear la aplicación que permitirá buscar y ver las películas disponibles. Las cosas principales son:

- **Inicio/cierre de sesión**. Por seguridad.
- **Listar películas**. Conectándose al API de Cloudflare Stream.
- **Ver película**. Que muestre los subtítulos e idealmente que tengan buena calidad.

**Más después de eso es irrelevante porque esto sería una herramienta personal ya que el contenido sería películas pirateadas**.

Con las cosas que hay que hacer ya un poco claras, creo que la UI la podría hacer con el siguiente stack:

- Ruby on Rails. Repositorio privado en GitLab.
- Hotwire/ViewComponent/Stimulus Reflex
- Tailwind CSS
- Devise
- PostgreSQL
- Heroku

El script para la carga de las películas lo tendría que escribir en Ruby o en Elixir.

