# Usando Pikapods

Para poder cargar o descargar archivos de los pods de Pikapods, hay que usar SFTP. Afortunadamente, no tengo que instalar ningún cliente gráfico, puedo usar la línea de comandos.

A continuación, algunos comandos básicos.

> [!Info]
> Los comandos salen de este tutorial https://linuxize.com/post/how-to-use-linux-sftp-command-to-transfer-files/

## Conectarse al pod

```bash
sftp p20907@coshifotos.pikapod.net
```

Y luego se provee la clave.

> [!Info]
> Las credenciales están en la configuración del Pod.

## Cargar Archivos y Carpetas

Una vez ya conectado al servidor SFTP

```bash
Connected to coshifotos.pikapod.net.
sftp>
```

El comando para subir archivos:
```bash
put filename
```

**Importante**: aquí se sube un archivo de nombre `filename` que está en la carpeta, en local, desde donde se lanzó el comando `sftp`. **Si el archivo está en otra ubicación, hay que pasar la ruta absoluta**.

Para cargar carpetas:
```bash
put -r folder_name
```