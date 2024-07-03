# Instalación y Desinstalación de AWS CLI

## Para Instalar

La forma más fácil en Macos es descargar el archivo .pgk y seguir las instrucciones. GUI Installer en este enlace -> https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

## Para Desinstalar

Instrucciones aquí -> https://docs.aws.amazon.com/cli/latest/userguide/uninstall.html

### En Macos M1

Estos fueron los comandos que seguí.

Borré el archivo credentials.

```
rm ~/.aws/credentials
```

Luego listé dónde está el enlace simbólico y la carpeta de origen:

```
ls -l /usr/local/bin/aws
lrwxr-xr-x  1 root  wheel  28 Feb 18  2023 /usr/local/bin/aws -> /Users/francisco/aws-cli/aws
```

Con esto ya podemos pasar a borrar los enlaces y la carpeta de instalación:

```
sudo rm /usr/local/bin/aws
sudo rm /usr/local/bin/aws_completer

rm -rf /Users/francisco/aws-cli
```

Con eso queda desinstalado.