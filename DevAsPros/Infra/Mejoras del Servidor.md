# Mejorando Desempeño del Servidor

> [!Warn]
> Cuando abra Enlacito al público tendré que moverlo a su propio VPS.

Tengo un VPS básico de Linode.

- 1 CPU
- 1 GB de RAM
- 25 GB de almacenamiento

No necesito más porque soy el único usuario de mis aplicaciones. El servidor hospeda cuatro aplicaciones Ruby on Rails.

- CashFlow
- CoshiNotes
- Enlacito
- SuperMenu

Hay un problema. En ocasiones abrir varias pestañas de CoshiNotes puede ser muy lento o no abren. A veces el servidor se queda pegado cuando quiero hacer algo dentro de esto.

> [!Important]
> El problema es CoshiNotes que hace uso extensivo de ActionCable. Eso consume y necesita mucha memoria y proceso disponible porque mantiene una conexión WebSocket abierta apenas entro a cualquier tema.

Este alto consume luce así en htop:

![[000.init.dap_server.png]]

Swap a tope. Memoria casi a tope también.

## Mejoras de Configuración de Passenger

Para estas aplicaciones no se recomienda un VPS tan pequeño. En todo caso no le di alternativa a Gepeto y me dijo que cambiara los settings de passenger:

- [passenger_max_pool_size](https://www.phusionpassenger.com/docs/references/config_reference/nginx/#passenger_max_pool_size)
- [passenger_pool_idle_time](https://www.phusionpassenger.com/docs/references/config_reference/nginx/#passenger_pool_idle_time)

### pool_idle_time en 300

Ese es su valor por defecto pero lo agrego al archivo para que sea explicito para mí.

Explicación de los docs:
> The maximum number of seconds that an application process may be idle. That is, if an application process hasn't received any traffic after the given number of seconds, then it will be shutdown in order to conserve memory.

### max_pool_size de 2 ❌

> [!Warn]
> Esta configuración no sirvió por que CoshiNotes necesita varios procesos.

El valor por defecto es 6.

Empecé con ese valor por recomendación de Gepeto. Sirvió para controlar la RAM pero no sirve para CoshiNotes.

![[001.passenger.pool.2.png]]

CoshiNotes, al usar Turbo Streams en los comentarios abre una conexión WebSocket constante y eso necesita mucha memoria.

En palabras de Claudio:
> Cada tema mantiene una conexión WebSocket abierta que ocupa un proceso de Passenger completo, incluso con Redis.
>
> Con 1GB RAM:
>  - Sistema: ~300MB
>  - Redis: ~100MB
>  - Sidekiq: ~150MB
>  - Nginx: ~20MB
>  - Disponible para Rails: ~430MB
>  - Cada proceso Rails: ~150-200MB
>
> Con 3 procesos = 450-600MB → estás al límite

### max_pool_size de 5 ❌

> [!Warn]
> Esta configuración no sirvió por que CoshiNotes necesita varios procesos.

Lo cambio a 5 para que CoshiNotes funcione como corresponde.

Así quedó la configuración al final:
```bash
cat /etc/nginx/conf.d/mod-http-passenger.conf
### Begin automatically installed Phusion Passenger config snippet ###
passenger_root /usr/lib/ruby/vendor_ruby/phusion_passenger/locations.ini;
passenger_ruby /usr/bin/passenger_free_ruby;

passenger_instance_registry_dir /var/run/passenger-instreg;

### End automatically installed Phusion Passenger config snippet ###

passenger_max_pool_size 5;

# Mantener 2 procesos siempre listos
passenger_min_instances 2;

passenger_pool_idle_time 300;

# Reciclar procesos para liberar memoria
passenger_max_requests 500;
```

### Prueba de `free -h`

Cuando ejecuto este comando veo algo aceptable:
```bash
free -h
              total        used        free      shared  buff/cache   available
Mem:          944Mi       668Mi       128Mi       0.0Ki       147Mi       135Mi
Swap:         511Mi       392Mi       119Mi
```

### max_pool_size de 6 y lo demás como estaba 🟢

Al final, para poder usar CoshiNotes con tranquilidad (abrir dos pestañas en escritorio y una en el móvil) me tocó dejar la configuración así:
```bash
cat /etc/nginx/conf.d/mod-http-passenger.conf
### Begin automatically installed Phusion Passenger config snippet ###
passenger_root /usr/lib/ruby/vendor_ruby/phusion_passenger/locations.ini;
passenger_ruby /usr/bin/passenger_free_ruby;

passenger_instance_registry_dir /var/run/passenger-instreg;

### End automatically installed Phusion Passenger config snippet ###

passenger_max_pool_size 6;

# Mantener 2 procesos siempre listos
# passenger_min_instances 2;

passenger_pool_idle_time 300;

# Reciclar procesos para liberar memoria
passenger_max_requests 500;
```


## Agregar Más Swap

### ¿Para qué?

Swap ayuda a que el servidor no explote (OOM).

Según gepeto:

Con poco swap:
- el kernel mata procesos Rails/Sidekiq
- Passenger/Puma “mueren” sin aviso

Con más swap:
- el sistema **sobrevive**
- los procesos no se caen

### Agregando más swap

Para un VPS de 1GB de RAM lo ideal es un swap del doble.

```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
```

Hacerlo persistente incluso después de reiniciar:
```bash
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

# Desinstalar Mega

Ya que iba a usar rclone + Cloudflare R2 decidí eliminar Mega por completo. Estos son todos los comandos:
```bash
rm -rf ./.megaCmd
sudo apt remove --purge megacmd

sudo apt purge megacmd

grep -r "mega.nz" /etc/apt/sources.list /etc/apt/sources.list.d/

sudo rm /etc/apt/sources.list.d/megasync.list /etc/apt/sources.list.d/megasync.list.save


sudo rm /usr/share/keyrings/meganz-archive-keyring.gpg
```

## Mejoras de Consumo Post Desinstalación

Después de desinstalar Mega, al revisar htop, me doy cuenta que el consumo de RAM no llega más al tope.

¿Podría estar esto ligado? ¿Ayuda que haya incrementado el swap?

![[002.htop.post.mega.remove.png]]

Fue haber quitado mega-cmd. Esta programa tenía un servicio activo corriendo. Eso le restaba RAM al servidor.

### free -h sin mega-cmd

La mejor evidencia es ver la salida de `free -h` arriba [[Mejoras del Servidor#Prueba de `free -h`]] y comparar con esta salida de hoy, 28 de Enero:
```
              total        used        free      shared  buff/cache   available
Mem:          944Mi       564Mi       144Mi       0.0Ki       235Mi       239Mi
Swap:         2.5Gi       891Mi       1.6Gi
```

Hay más RAM disponible.