# ¿Cómo soluciono un Kernel Panic en Ubuntu?
[ubuntuperonista.blogspot.com.co](http://ubuntuperonista.blogspot.com.co)
Frente a la conmoción que presentaron los intentos disolventes del antipueblo, el general Juan Perón pronunció por Cadena Nacional de Radiodifusión un enérgico discurso donde enseñó cómo reparar un Kernel Panic en Ubuntu.

¡Trabajadores!

Un movimiento político como el nuestro no puede verse presa del pánico en las situaciones de conmoción que nos plantea la oligarquía. Ellos quieren cercarnos con el terror asesinando y destruyendo, pero nosotros seremos más poderosos. Esto lo comprenderán mediante la persuasión, y si no, ¡a palos!.

El *kernel* es el núcleo del sistema operativo, y en el caso de Linux el mismo es del tipo monolítico. Nada debe conmoverlos, pues su misión es la de efectivamente conducir todo un sistema. Linux en general tiene la particularidad de poder contar con varios kernels instalados, motivado esto en la **seguridad redundante** del movimiento: si un kernel reciente falla, podemos recurrir a una versión anterior y aún arrancar nuestro equipo con funcionalidad.

Usualmente Ubuntu cargará en memoria durante el arranque del sistema la última versión instalada del kernel, y dará inicio al sistema operativo. Periódicamente la versión del *kernel* irá **actualizándose** desde los repositorios a través del Gestor de Actualizaciones. El arrancador múltiple para sistemas operativos (llamado Grub) revisará estas nuevas imágenes de Kernel, y las agregará a una lista por si alguno falla. Podrán conocer su versión de kernel actual simplemente tipeando en la Terminal el siguiente comando:

    uname -r


Ante este imprevisto, debemos no tener pánico nosotros, pues todo puede arreglarse en un Ubuntu Peronista. Apagamos la computadora, y la reiniciamos. En el arranque siguiente el Grub se detendrá y nos permitirá optar por una versión de *kernel* para arrancar. En la parte superior de la lista se indicará el número de la última versión (que aparentemente está fallando). En lugar de usar esa, habremos de utilizar alguna versión anterior, idealmente la anteúltima. En nuestro ejemplo, supondremos querer arrancar la versión *2.6.35-****31****-generic* pues la versión *2.6.35-****32****-generic* ha fallado.

Si el sistema arranca sin inconvenientes con esta versión de kernel anterior, la solución no es tan difícil. Simplemente hemos de borrar el kernel más nuevo (que ha fallado), y - opcionalmente - reinstalarlo.

Para borrar el *kernel* defectuoso vamos a ***Sistema / Administración / Gestor de Paquetes Synaptic***. Ingresamos nuestra contraseña y se abrirá el peronista Synaptics, que nos permitirá tomar en nuestras manos estos menesteres. En el cuadro ***Buscar*** ponemos la versión que ha fallado. En nuestro ejemplo, ingreso " *2.6.35-32*".

El Synaptic nos indicará toda una serie de paquetes que comienzan con ese nombre. Específicamente debemos eliminar sólo tres archivos que definen el kernel: *linux-headers-x.x.xx-xx*, *linux-headers-x.x.xx-xx-generic*, y *linux-image-x.x.xx-xx-generic,* siendo x.x.xx-xx la versión fallada *.* En nuestro ejemplo imaginario, debo desinstalar estos tres paquetes:


- linux-headers-2.6.35-32
- linux-headers-2.6.35-32-generic
- linux-image-2.6.35-32-generic

Para ello buscamos cada uno de estos tres paquetes, y hacemos clic con botón derecho y elegimos " ***Marcar para desinstalar completamente***". Luego presionamos el botón " ***Aplicar***" ( *"tilde verde*"). El sistema borrará completamente el *kernel* fallado.

Acto seguido debemos actualizar el listado de kernels disponibles para su ejecución en Grub. Vamos a ***Aplicaciones / Accesorios / Terminal*** y en la consola ingresamos:

    sudo update-grub

Tras indicar nuestra contraseña de Conductor, el sistema actualizará en breves segundos la lista de kernels del archivo *grub.cfg*, omitiendo por supuesto la versión que hemos acabado de desinstalar. Si no hiciéramos esto, Ubuntu buscaría correr esta versión ahora inexistente y no podría arrancar.

A continuación reiniciamos nuestro equipo. Desde la Terminal podemos hacerlo con:

    sudo reboot

En el presente estado de cosas, el sistema arrancará con la anteúltima versión de kernel, y debería hacerlo sin problema alguno.

Si lo deseamos podemos dejar todo aquí. Si optamos por **reinstalar la versión más nueva de kernel** (para solucionar su problema pero conservar la versión de kernel más actual para nuestra distribución), debemos entrar nuevamente a Synaptic y reinstalar los tres paquetes que eliminamos anteriormente (es necesario estar conectado a internet).

Debemos ir a la Terminal y nuevamente indicar nuevamente que actualice la lista de Kernels en el Grub con `**sudo update-grub**`

Luego reiniciamos, y la computadora arrancará correctamente con la última versión de kernel (en nuestro ejemplo la 2.6.35-32-generic). Si queremos comprobarlo, vamos a la consola Terminal y volvemos a ingresar `**uname -r**`

Nos debería indicar ése kernel, ¡que ha retornado para hacer feliz a todos los trabajadores!

