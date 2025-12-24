# NAS o DAS

¿Qué es lo siguiente que necesito para Coshinema, una NAS o un DAS?

## Motivación

Quiero ya poder usar mi servidor Plex. Para ello quiero poder conectar el almacenamiento al PC sin temor de que el cable USB sea movido y por lo tanto correr el riesgo del dañar el HDD al desconectarse mientras está siendo leído.

También me gustaría tener una forma menos engorrosa para hacer los respaldos entre Atronador y Quebrantahuesos que conectarlos por USB y pasar archivos de un disco a otro.

## Claves de un NAS

Network-Attached Storage.

Tienen un sistema operativo, CPU y RAM.

Detalles si me hago a una:
- Va conectado al router
- Va conectado a la corriente eléctrica 
- Necesitaría una UPS
	- Por los cortes de energía sería mejor protegerlo
- Necesitaría HDDs de 3.5"???

Para este lo ideal es usar los discos en una configuración RAID. Sin embargo, no me suena por el esfuerzo en mantenimiento y necesitar muchos discos.


## Claves de un DAS

Direct-Attached Storage.

Es básicamente la configuración que tengo ahora mismo. Un HDD en una carcasa conectado por USB al computador. Sin embargo, la clave está en que hay carcasas más avanzadas que aportan velocidad y permiten otras configuraciones aprovechando varias bahías de discos.

Este me interesa porque podría tener una configuración un poco más avanzada usando software como [SnapRAID](https://www.snapraid.it/) y [MergerFS](https://github.com/trapexit/mergerfs).

## SnapRAID + mergerfs

SnapRAID sirve para crear un disco de paridad que almacena datos para facilitar la recuperación de datos en caso de falla de alguno de los discos del DAS.

> [!Note]
> SnapRAID necesita que el disco de paridad sea más grande que los discos de datos.

Por su parte MergerFS me permite combinar varios discos en un mismo "disco" así que puedo aprovechar el almacenamiento de varios discos de diferentes tamaños mediante una solo ruta de acceso.

# Posibles Configuraciones

## DAS  + SnapRAID + mergerfs (opcional)

Pongo los tres discos en el DAS y reparto el contenido de esta forma:

- Atronador: películas, series y anime
- Quebrantahuesos: vídeos de YT
- Embestidor u otro: disco de paridad para SnapRAID

Al poner los discos en la carcasa podré acceder al contenido de todos con una sola conexión.

### mergerfs

Opcionalmente, configuro mergerfs para crear un solo volumen para simplificar el manejo. Donde cada disco se mantendría por separado con sus respectivas carpetas de multimedia diferente. Cuando baje nuevas películas o vídeos de YT copio a la respectiva carpeta y se mantendrán en cada disco que corresponde.

Crear un solo volumen con mergerfs solo sería una conveniencia.