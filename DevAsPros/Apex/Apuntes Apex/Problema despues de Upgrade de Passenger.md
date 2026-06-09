# Super Menu y Postlane caídos después de upgrade de passenger

> [!Info]
> Al final fue un problema con Passenger. Se desactivó todo lo de passenger y se hizo una petición http y nginx la respondía.
> 
> Empecé el cambio a Puma.

## Versión de Passenger es 6.1.2 y nginx 1.24.0

Lo puedo ver en el ping que tiré días antes:
```
curl -I https://supermenu.devaspros.com/ping

HTTP/1.1 200 OK
Content-Type: text/html
Connection: keep-alive
Status: 200 OK

X-Powered-By: Phusion Passenger(R) 6.1.2
Server: nginx/1.24.0 + Phusion Passenger(R) 6.1.2
```

> [!Note]
> Aunque DeepSeek me hizo reinstalar, la versión que siempre ha dado el problema es la 6.1.2

## DeepSeek me hizo dar un paso en falso ℹ️

Dijo que esto:
```bash
apt list --installed | grep passenger

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

libnginx-mod-http-passenger/noble,now 1:6.1.2-1~noble1 amd64 [installed,upgradable to: 1:6.1.4-1~noble1]
passenger/noble,now 1:6.1.2-1~noble1 amd64 [installed,upgradable to: 1:6.1.4-1~noble1]
```

Significaba que hubo un intento de actualización y no se completó. Así que hice varias cosas para reinstalar passenger. Cosas que no corrigieron el problema.

Cuando lo confronté admitió que no era un "upgrade sin terminar":
> Tienes toda la razón. **"Upgradable"** solo significa que hay una versión más nueva disponible en los repositorios, **no** que se haya intentado instalar. Fue un falso diagnóstico de mi parte. La reinstalación fue un paso innecesario que desvió la solución.

# Los Errores 🐞

## worker process exited on signal 11 (core dumped)

Veo esto:
```bash
sudo tail -f /var/log/nginx/error.log

2026/06/09 16:29:03 [alert] 216753#216753: worker process 217057 exited on signal 11 (core dumped)
2026/06/09 16:29:07 [alert] 216753#216753: worker process 217099 exited on signal 11 (core dumped)
```