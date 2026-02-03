# Configuración de SnapRAID

SnapRAID es una alternativa a RAID para hace respaldos de discos duros en array. O sea, la configuración que tengo con el DAS.

> [!Note]
> Al decidir usar el DAS tenía que completar la configuración con SnapRAID.
> 
> Ver [[NAS o DAS 🟢]]

Ver sitio web: https://www.snapraid.it/

## Instalación

Estos fueron los pasos que seguí después de bajar el archivo tar de la página web.

```bash
tar xzvf snapraid-13.0.tar.gz
cd snapraid-13.0/
./configure
make
make check
make install
```

Luego de copiar el archivo conf borré la carpeta y el archivo tar:
```bash
rm -rf snapraid*
```

Por recomendación del sitio "havetheknowhow" mandé el archivo conf a `/media/cesc/snapraid/` y de ahí hice un symlink a `/etc/`

```bash
sudo mkdir -p /media/cesc/snapraid
cp ~/Descargas/snapraid-13.0/snapraid.conf.example /media/cesc/snapraid/snapraid.conf
```

Finalmente, hice el symlink:
```bash
sudo ln -s /media/cesc/snapraid/snapraid.conf /etc/snapraid.conf
```

## Configuración de snapraid.conf

El tutorial en "diymediaserver" fue el mejor para lograr esto. Explicó todo.

El resultado final luce así:
```bash
parity /media/cesc/Vigia/snapraid.parity

content /media/cesc/Vigia/snapraid.content
content /media/cesc/Atronador/snapraid.content
content /media/cesc/Rapaz/snapraid.content

data d1 content /media/cesc/Atronador/
data d2 content /media/cesc/Rapaz/
```

Lo demás quedó según como está en el archivo copiado al bajar el software.

## Prueba

Comando `snapraid status`

## Puesta en Marcha

Comando `snapraid sync`

## Recursos

Los tutoriales de donde saqué la info.

- De [Setting up SnapRAID on Ubuntu to Create a Flexible Home Media Fileserver](https://zackreed.me/setting-up-snapraid-on-ubuntu/) saqué los pasos para la instalación.
- De [How to configure SnapRAID 11.x on Ubuntu Server](https://havetheknowhow.com/configure-the-server/configure-snapraid/) saqué el dato de tener la configuración en `/media/cesc` y symlinkear a `/etc/`.
- Finalmente, de [Master the Basics - How to Install and Use SnapRAID for a Resilient Home Media Server](https://diymediaserver.com/post/master-the-basics-how-to-install-snapraid/) fue de donde entendí toda la movida y guié la configuración para mi entorno.