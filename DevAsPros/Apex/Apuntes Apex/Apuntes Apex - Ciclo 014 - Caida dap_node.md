# Apuntes Apex - Ciclo 014 - Caída de DAP Node

## Caída del VPS: 2026-11-18

### Resumen

- **Afectadas:** enlacito.co, cashflow, coshinotes (mismo VPS en Linode, 1 GB de RAM).
- **Ventana:** ~13:30–15:15 UTC (8:30–10:15 Bogotá). Fallos intermitentes, no una caída limpia.
- **Síntomas:** Cronitor "Timed Out" en `/ping`; en el navegador, `502 Bad Gateway` en las tres apps.
- **Causa:** la tabla de conexiones del kernel (`nf_conntrack`) se llenó (7680/7680). El kernel descartaba todo paquete de conexión nueva: entrante, saliente y de localhost.
- **Arreglo:** subir el límite de `conntrack` y bajar sus timeouts con `sysctl -w`. Recuperación inmediata.

### Qué es conntrack y por qué se llenó

- El firewall del kernel (ufw/iptables) lleva una tabla con cada conexión. Toda conexión nueva necesita un lugar en ella. Si no hay espacio, el kernel descarta el paquete en silencio.
- El límite era 7680, calculado a partir de la RAM (muy bajo con 1 GB).
- Por defecto, una conexión "establecida" que nunca se cerró bien ocupa su entrada 5 días.
- Los scanners que abren conexiones y las abandonan la llenaron en ~7 h desde el reinicio de las 06:01 UTC.

### Cómo produjo cada síntoma

- **Cronitor "Timed Out":** su conexión entrante se descartaba.
- **502 en las tres apps:** nginx no lograba conectarse con Puma ni siquiera por localhost.
- **DNS y Geocoder fallando:** las conexiones salientes también se descartaban.
- **Intermitente:** al caducar alguna entrada, unas pocas conexiones pasaban hasta que se volvía a llenar.

### Evidencia

#### Descartado

**Cronitor:** `Timed Out`, sin código HTTP.
→ Si Puma hubiera estado caído con nginx vivo, habría salido un 502 inmediato. Descarta un proceso caído y un problema de certificado.

**Navegador:** 502 en las tres apps.
→ El problema es de la máquina, no de una app.

`swapon --show`
```
/swapfile  2G   207.1M
/dev/sdb   512M 219.2M
```
→ Hay swap y no está llena.

`sudo journalctl -k --since "2026-11-18 00:00" | grep -iE "oom|killed process"`
Salida vacía.
→ El kernel no mató procesos por falta de memoria.

`vmstat 1 5`
```
si 0  so 0  wa 0  st 0  id 83–93
```
→ Sin movimiento de swap, sin espera de disco, sin CPU robada por el proveedor, CPU casi libre. Descarta la memoria.

`uptime`
```
load average: 0.31, 0.26, 0.21
```
→ Máquina desocupada.

`ss -ltn | grep -E ':(3001|3003|3005)'`
```
LISTEN  0  1024  0.0.0.0:3005
```
→ Recv-Q = 0: Puma no estaba saturado; las conexiones no le llegaban.

Requests por minuto en los access logs de nginx: entre 1 y 59 sumando las tres apps.
→ Descarta un ataque de tráfico.

#### Apuntando a la red

`sudo tail -20 /var/log/nginx/enlacito.error.log`
```
upstream timed out (110: Connection timed out) while connecting to upstream ... "http://127.0.0.1:3005/links"
no live upstreams while connecting to upstream
```
→ nginx no lograba abrir una conexión TCP con Puma en localhost, con la CPU libre: se perdían paquetes dentro de la máquina.
→ El contador de conexiones de nginx subía miles por minuto, muy por encima de los requests reales.

Top IPs, user agents y rutas: `40.160.226.237` (334), `Go-http-client/1.1` (213), ruta `*` (160).
→ Scanners que abren conexiones sin hacer requests válidos.

`sudo journalctl -t enlacito_puma --since "2026-11-18 13:00"`
```
Geocoding API not responding fast enough
Geocoding API connection cannot be established
```
→ Puma estaba vivo, pero el servidor no lograba conectarse hacia fuera.

`last -x reboot`
```
Wed Nov 18 06:01 / Wed Nov 11 06:01 / Thu Oct 29 06:01
```
→ Reinicios de `unattended-upgrades`. No es la causa, pero explica que la tabla estuviera vacía a las 06:01 y llena a las 13:30.

#### Confirmación

`sudo dmesg -T | grep -iE "conntrack|table full|dropping|nf_"`
```
nf_conntrack: nf_conntrack: table full, dropping packet
```
→ La causa, dicha tal cual por el kernel.

`cat /proc/sys/net/netfilter/nf_conntrack_count /proc/sys/net/netfilter/nf_conntrack_max`
```
7680
7680
```
→ Tabla al 100 %.

`ss -s`
```
estab 384, timewait 230, orphaned 50
```
→ ~670 sockets reales frente a 7680 entradas: la tabla estaba llena de conexiones que ya no existían.

`time getent hosts ipinfo.io` → `real 0m5.023s`
`curl -sS -m 10 https://ipinfo.io/8.8.8.8` → `Could not resolve host: ipinfo.io`
→ DNS y salida a internet rotos.

Después de aplicar el `sysctl -w`, las tres apps se recuperaron al instante.

### Arreglo aplicado (temporal)

```bash
sudo sysctl -w net.netfilter.nf_conntrack_max=65536
sudo sysctl -w net.netfilter.nf_conntrack_tcp_timeout_established=3600
sudo sysctl -w net.netfilter.nf_conntrack_tcp_timeout_time_wait=30
```

| Ajuste | Antes | Ahora | Efecto |
|---|---|---|---|
| `nf_conntrack_max` | 7680 | 65536 | Caben 8 veces más conexiones (~20 MB de RAM como máximo) |
| `tcp_timeout_established` | 5 días | 1 hora | Las conexiones abandonadas liberan su lugar mucho antes |
| `tcp_timeout_time_wait` | 120 s | 30 s | Las conexiones ya cerradas se borran más rápido |

### Pendientes

#### 1. Hacerlo permanente 🟢

`sysctl -w` se pierde al reiniciar, y `unattended-upgrades` reinicia cuando hay upgrades de kernel.

```bash
echo nf_conntrack | sudo tee /etc/modules-load.d/conntrack.conf
sudo tee /etc/sysctl.d/99-conntrack.conf <<'EOF'
net.netfilter.nf_conntrack_max = 65536
net.netfilter.nf_conntrack_tcp_timeout_established = 3600
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 30
EOF
sudo sysctl --system | grep conntrack
```

El `modules-load.d` es necesario: sin él, al arrancar los sysctl se aplican antes de que cargue `nf_conntrack` y se pierden en silencio.

#### 2. Vigilar y ver quién llena la tabla 🟡

```bash
# Entradas ocupadas (si sube sin parar hacia 65536, hay abuso)
cat /proc/sys/net/netfilter/nf_conntrack_count

# Por IP de origen
sudo grep -oE "src=[0-9.]+" /proc/net/nf_conntrack | awk 'NR%2==1' | sort | uniq -c | sort -rn | head -10

# Por estado TCP
sudo awk '$3=="tcp"{print $6}' /proc/net/nf_conntrack | sort | uniq -c | sort -rn
```

#### 3. Cambios en el repo de enlacito 🟡

- **Geocoder fuera del request:** `AnalyticsService` llama a `Geocoder.search` dos veces, en vivo, en cada redirección. Si la API externa se pone lenta, ocupa los 3 threads de Puma. Moverlo a `ProcessAnalyticsJob`, con una sola llamada y un timeout corto.
- **nginx con `127.0.0.1` en vez de `localhost`** (las tres apps): `proxy_pass http://localhost:PUERTO` hace que nginx trate `127.0.0.1` y `[::1]` como dos servidores. Puma solo escucha en IPv4, y basta un fallo en cada uno para que nginx devuelva "no live upstreams", o sea 502 para todos durante 10 s. Cambiar a `proxy_pass http://127.0.0.1:PUERTO;`.

### Si vuelve a pasar ℹ️

Revisar primero:
```bash
sudo dmesg -T | grep -i conntrack | tail
cat /proc/sys/net/netfilter/nf_conntrack_count /proc/sys/net/netfilter/nf_conntrack_max
```