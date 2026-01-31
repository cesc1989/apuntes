# YTtoPlex - De YouTube a Plex

Idea de herramienta local para descargar vídeos de YouTube, organizarlos, copiar al HDD (Atronador) y luego a re-escanear la librería en Plex. Solo tendría que pasarle la URL del vídeo o playlist y la herramienta hará el resto.

## Concepción del Problema

En el ThinkPad suelo ver vídeos de YT usando FreeTube. Esta herramienta funciona muy bien en general pero cuando YT hace alguna actualización suele dejarla inútil. Suelen arreglar los problemas rápido pero mientras tanto me toca usar el modo embed o usar el mismo YT en Firefox.

Quisiera cambiar esa dependencia en FreeTube y tener algo menos afectado por los cambios de YT y que pueda controlar más.

También quisiera algo que me permitiera ver vídeos en el TV. Para lograrlo necesito descargarlos y montarlos en Plex.

En definitiva FreeTube funciona bien pero están en una carrera contra Google así que siempre habrá este tipo de problemas. Hay claramente una restricción de espacio en disco disponible para esto así que no lo veo como algo permanente en términos generales sino para casos puntuales o de vídeos que quiera ver prestando atención (LMI, Wendover, Conferencias).

## Idea

Una interfaz (web o CLI) que le pueda dar una URL de YT, sea de vídeo o de playlist, y descargué el contenido (calidad 1080p), lo organice (nombre, carpeta), copie al HDD (Atronador) y finalmente haga un re-escaneo de la librería correspondiente en Plex.

Lo podría resumir en un flujo como:

- Pasar URL a CLI
- CLI descarga usando yt-dlp
- Organizar archivo (renombrar, carpeta)
- Copiar al HDD (cp a /media/cesc/Atronador)
- Re-escanear librería Plex

Para empezar la herramienta podrá ser un CLI. Sea que lo haga en Elixir/Ruby/Bash la idea es poder configurarla de tal forma que se ejecute pasándole todo lo necesario para hacer su trabajo. Debe poder referenciar diferentes carpetas e identificadores (para las librerías de Plex). Probablemente todo mediante banderas.

> [!Note]
> La última vez que revisé no había forma de empaquetar un programa en Elixir como binario. En Ruby la cosa es similar.
>
> En todo caso no veo mal una herramienta que ejecute como `ruby X`. Solo será para uso en el servidor Plex.


### Dependencias

Las cosas que necesito tener en el sistema para lograr esto y/o temas que necesito aclarar antes de hacer el CLI.

> [!Important]
> Descartada cualquier idea de usar Elixir como lenguaje para el CLI. Ver [Respuesta Jose Valim](https://elixirforum.com/t/how-suitable-is-elixir-for-a-cli-application/36184/13).

- yt-dlp: https://github.com/yt-dlp/yt-dlp
	- ~~Tentativa: librería en Elixir: https://github.com/kevinetore/youtube-dl.ex~~
- Plex: Escanear vs Refrescar: https://support.plex.tv/articles/200289306-scanning-vs-refreshing-a-library/
- PMS Server Commands: https://support.plex.tv/articles/201638786-plex-media-server-url-commands/
- Plextopedia: Scan library: https://www.plexopedia.com/plex-media-server/general/scan-library/

# Desarrollo

## 7 de Octubre, 2025

Con ayuda de DeepSeek hice la 1era versión. Copié el script a mis [dotfiles](https://github.com/cesc1989/dotfiles/commit/43c62a0305f65d8b464da02b66728c5cf0248c47). Se ejecuta así:
```bash
bash ./extras/yttoplex.sh -u https://www.youtube.com/playlist?list=PLd0qxsHDZqYln-sfR7cXmdZxd2H6J7vwM \
  -p esplada_y_core \
  -d "$HOME/yttoplex-destino"
```

Para ver la ayuda:
```bash
bash ./extras/yttoplex.sh
```