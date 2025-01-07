# Sistema de Archivos y Permisos en HDD externo para Plex

Sobre el format de archivo: https://www.reddit.com/r/PleX/comments/gewrgk/hard_drive_format_for_plex_servers/

Si el servidor Plex va a correr en Windows, el formato de archivos debe ser NTFS.

Si el servidor Plex va a correr en Linux, el formato de archivos debe ser ext4.

# Disco con format ext4

Formatee el disco a este formato, copié una carpeta de películas y sí funcionaron los comandos que probé en el formato anterior.

    sudo chgrp plex /media/cesc/
    sudo chmod g+rX /media/cesc/
    
    sudo chgrp plex /media/cesc/Atronador/
    sudo chmod g+rX /media/cesc/Atronador/

Esta es una guía “oficial” en el foro de Plex. No la usé pero dejó aquí el enlace por si las moscas. https://forums.plex.tv/t/using-ext-ntfs-or-other-format-drives-internal-or-external-on-linux/198544

# Disco con formato FAT

Probé estos comandos de [esta respuesta](https://askubuntu.com/a/515595/167553) en AskUbuntu.

Primero confirmé que el disco se monté en esta ruta:

    /media/<user>/<HDD Name>
    
    /media/cesc/Atronador

Agregar a mi usuario de sistema al grupo plex

    sudo adduser "$USER" plex

Y ahora cambio de permisos

    sudo chgrp plex /media/cesc/
    sudo chmod g+rX /media/cesc/
    
    # sudo chgrp plex /media/cesc/Atronador/ -> Este falló
    sudo chmod g+rX /media/cesc/Atronador/
    
    sudo setfacl -m g:plex:rx /media/cesc

Sin embargo, este comando de cambio de grupo falló:

    $ sudo chgrp plex /media/cesc/Atronador/
    chgrp: cambiando el grupo de '/media/cesc/Atronador/': Operación no permitida

¿Por qué no se puede? Según explican en [esta respuesta](https://superuser.com/a/57096/372807), el sistema de archivos FAT no tiene soporte para permisos.


> Linux already has FAT support for read & write although **they do not support permissions**.

Así aparece el sistema de archivos del HDD

    $ df -T
    S.ficheros     Tipo  bloques de 1K    Usados Disponibles Uso% Montado en
    /dev/sda2      ext4       85929064  12847804    68670332  16% /
    /dev/sda4      ext4      135150272  19800220   108411940  16% /home
    /dev/sda1      vfat         291224      6220      285004   3% /boot/efi
    /dev/sdc1      vfat      976522304 304114560   672407744  32% /media/cesc/Atronador

En otra respuesta también dicen esto al respecto de FAT:

> It's probably formatted as FAT, which doesn't support file permissions. Use ext3 instead.



