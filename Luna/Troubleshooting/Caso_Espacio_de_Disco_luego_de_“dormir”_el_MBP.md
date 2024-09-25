# Caso Espacio de Disco luego de “dormir” el MBP

# Conclusión

Nada funcionó. La única solución viable era formatear el equipo.

----------

Poner a dormir el macbook resulta en que se gasta mucho espacio de disco. Puede estar así durante el uso normal:

![[101.mbp.intel.disk.issue.png]]

Y luego de dormir puede pasar a menos de 40GB.

La solución que encuentro es reiniciar el MBP pero últimamente ha estado pasando incluso al haber reiniciado el día anterior.

# Otras Soluciones

## Limitar el número de filas de la terminal

Lo primero fue limitar el número de filas que tiene la terminal luego de ver y buscar sobre este mensaje de error:

![[102.intel.mbp.low.storage.png]]

> Your disk space is critically low.
> 
> Older terminal scrollback contents may be automatically discarded to conserve VM backing store.

Se cambia en esta sección:

![[103.change.terminal.config.png]]

~~Al parecer esta fue la solución~~. Si lo es tendría sentido ya que tengo muchas ventanas de terminal abiertas. Veamos:

- forms: 4 pestañas
- therapist: 4 pestañas
- dashboard: 4 pestañas

Si cada una mantuviera un registro de 10.000 filas entonces sería 10 * 12. Ahí habría un serio problema de mantener almacenado todo eso.

## Revisar el espacio que consume el poner a dormir

También encontré en [esta pregunta](https://apple.stackexchange.com/questions/195967/mac-os-x-10-10-3-running-out-of-disk-space-after-sleep-potential-memory-leak) sobre revisar el tamaño de la imagen de “dormir” que genera mac. Supuestamente el modo “dormir” lo que hace es pasar lo de la RAM a disco entonces eso podría consumir espacio.

El comando:
```bash
ls -lh /private/var/vm/
total 6291456
-rw------T  1 root  wheel   1.0G Nov 18 07:00 sleepimage
-rw-------  1 root  wheel   1.0G Nov 17 12:05 swapfile0
-rw-------  1 root  wheel   1.0G Nov 17 20:43 swapfile1
```

```bash
ls -lh /private/var/vm/sleepimage
-rw------T  1 root  wheel   1.0G Dec  7 07:10 /private/var/vm/sleepimage
```

Enlaces sobre sleepimage:

- https://osxdaily.com/2010/10/11/sleepimage-mac/

## Aplicación de gestión de espacio

Dice que lo que más ocupa está en Documents pero luego se ve un valor muy inferior.

![[104.mbp.intel.espacio.png]]

Un [hilo de comentan](https://forums.macrumors.com/threads/hard-drive-space-being-slowly-eaten-then-freed-at-restart.941226/) muchas cosas pero igual no se llega a la conclusión más que formatear.

