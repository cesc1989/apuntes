# Instalación de Software en Linux Mint

## Delta

> A syntax-highlighting pager for git, diff, and grep output.

Sitio web: https://github.com/dandavison/delta/

### Instrucciones

Hay que descargar el archivo .deb en https://github.com/dandavison/delta/releases

Es el que tiene nombre similar a:
```
git-delta_0.18.2_amd64.deb
```

Una vez descargado el comando para instalar es:
```
sudo dpkg -i git-delta_0.18.2_amd64.deb
```

Se puede comprobar con:
```
which delta
/usr/bin/delta

delta --version
delta 0.18.2
```

