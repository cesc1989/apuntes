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

Así terminó el proceso después de 10 horas:
```bash
Self test...
Loading state from /media/cesc/Vigia/snapraid.content...
WARNING! Content file '/media/cesc/Vigia/snapraid.content' not found, attempting with another copy...
Loading state from /media/cesc/Atronador/snapraid.content...
WARNING! Content file '/media/cesc/Atronador/snapraid.content' not found, attempting with another copy...
Loading state from /media/cesc/Rapaz/snapraid.content...
No content file found. Assuming empty.
Scanning...
Scanned d2 in 1 seconds
Scanned d1 in 1 seconds
Using 38 MiB of memory for the file-system.
Initializing...
Resizing...
Saving state to /media/cesc/Vigia/snapraid.content...
Saving state to /media/cesc/Atronador/snapraid.content...
Saving state to /media/cesc/Rapaz/snapraid.content...

     d1  0% | 
     d2  0% | 
 parity 98% | ******************************************************************
   raid  0% | 
   hash  0% | 
  sched  0% | 
   misc  0% | 
            |____________________________________________________________________
                              wait time (total, less is better)

Everything OK
```

## Claves Uso y Configuración de SnapRAID

### snapraid.parity

- Los ==datos de paridad se guardan en un disco de paridad dedicado==. Este disco debe ser independiente de los discos de datos.
- Cada que se agreguen más datos en los discos hay que correr `snapraid sync`.
- A **mayor cantidad de discos de paridad, mayor cantidad de discos se pueden rescatar**.
- Un disco de paridad puede proteger multiples discos de datos.
- El disco de paridad debe tener suficiente espacio para contener la información de paridad.

### snapraid.content

- La configuración de *parity* y *content* van de la mano.
	- Se pueden usar los mismos discos de datos como discos de `content`.
- Se pueden usar los discos de datos como discos de `content`.

### snapraid data

- Estos son los discos duros o cualquier almacenamiento.
- En el archivo conf hay que definir: *parity, content y data* para que snapraid trabaje como se debe.
- Los datos/contenido debe estar separado en varios discos para mejor organización y redundancia.

## Recursos

Los tutoriales de donde saqué la info.

- De [Setting up SnapRAID on Ubuntu to Create a Flexible Home Media Fileserver](https://zackreed.me/setting-up-snapraid-on-ubuntu/) saqué los pasos para la instalación.
- De [How to configure SnapRAID 11.x on Ubuntu Server](https://havetheknowhow.com/configure-the-server/configure-snapraid/) saqué el dato de tener la configuración en `/media/cesc` y symlinkear a `/etc/`.
- Finalmente, de [Master the Basics - How to Install and Use SnapRAID for a Resilient Home Media Server](https://diymediaserver.com/post/master-the-basics-how-to-install-snapraid/) fue de donde entendí toda la movida y guié la configuración para mi entorno.