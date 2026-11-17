# Usando Homer para Dashboard Personal de Enlaces

Tenía esta idea de un espacio con todos los enlaces de interés. Me encuentro muchas veces abriendo las mismas URLs o algunas variantes (ejemplo, las de github). Entonces pensé porqué no tener un tablero con los enlaces organizados por grupos para así acceder más fácil.

Descarté la página inicial de Firefox porque se van cambiando a medida que se usen o no. Quería algo más fijo. Encontré Homer.

Website: https://github.com/homer-dashboard/homer

> A dead simple static **HOM**epage for your serv**ER** to keep your services on hand, from a simple `yaml` configuration file.

## Lanzar y configurar

Con este comando lo lancé en el pc Coshinema:
```bash
docker run -d \
  --name homer \
  -p 8080:8080 \
  -v "$HOME/homer/assets:/www/assets" \
  --restart=unless-stopped \
  b4bz/homer:latest
```

Tuve antes que crear la carpeta `"$HOME/homer/assets"` que es donde irá el archivo de configuración e imágenes.

Una vez personalizado el archivo quedó así:
![[Pasted image 20261117163639.png]]

El archivo está en la carpeta `~/Documents/Documents/Personal/Homer/` en el MBP. También sincronizada a Mega para respaldo.

> [!Info]
> Para ver todas las opciones de configuración: https://github.com/homer-dashboard/homer/blob/main/docs/configuration.md

## Detener y borrar el container

Para detener:
```bash
docker stop homer
```


Para borrar, en caso de que se necesite reconstruir:
```bash
docker rm homer
```