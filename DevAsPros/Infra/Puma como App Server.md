# Usar Puma como App Server

## Notas

> [!Importante]
> Hay que poner una instrucción que haga `nginx reload` al final del paso 4. Tiene que ser aquí porque es en este paso donde se copian los archivos nuevos a la carpeta de la app en el VPS.

```bash
# Reload nginx para que tome cualquier cambio en la config
#
echo "$(date '+%F %T') Reloading nginx" >> $LOG_DIR/211_nginx_reload.log 2>&1
sudo systemctl reload nginx >> $LOG_DIR/211_nginx_reload.log 2>&1
```

## Super Menú

### config/puma.rb

```ruby
threads_count = ENV.fetch("RAILS_MAX_THREADS", 3)
threads threads_count, threads_count

port ENV.fetch("PORT", 3006)

# Allow puma to be restarted by `bin/rails restart` command.
plugin :tmp_restart

# Specify the PID file. Defaults to tmp/pids/server.pid in development.
# In other environments, only set the PID file if requested.
pidfile ENV["PIDFILE"] if ENV["PIDFILE"]
```

### supermenu.puma.service

```bash
/etc/systemd/system/supermenu.puma.service
```

Configuración:
```bash
[Unit]
Description=supermenu (puma)
After=network.target

[Service]
Type=simple

WorkingDirectory=/home/ubuntu/supermenu/app

# Igual que Sidekiq: memoria más estable en Ruby
Environment=MALLOC_ARENA_MAX=2

# Si tienes variables globales (DB, secrets, etc)
EnvironmentFile=/home/ubuntu/.supermenu.envs

ExecStart=/home/ubuntu/.rubies/ruby-3.4.8/bin/bundle exec puma -C config/puma.rb

ExecReload=/bin/kill -USR2 $MAINPID

Restart=on-failure
RestartSec=3

# Logs igual que Sidekiq (consistencia del VPS)
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=supermenu_puma

[Install]
WantedBy=multi-user.target
```

Se activa así:
```bash
sudo systemctl daemon-reload
sudo systemctl enable supermenu.puma.service
sudo systemctl restart supermenu.puma.service
```

Comprobación:
```bash
sudo systemctl status coshinotes.puma
```

### Configuración de Nginx

Así quedó el 9 de Junio:
```nginx
server {
  listen 80;
  server_name supermenu.devaspros.com;

  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl;
  server_name supermenu.devaspros.com;

  ssl_certificate /etc/letsencrypt/live/supermenu.devaspros.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/supermenu.devaspros.com/privkey.pem;

  # Logs
  access_log /var/log/nginx/supermenu.access.log;
  error_log /var/log/nginx/supermenu.error.log info;

  # Rails public assets
  root /home/ubuntu/supermenu/app/public;

  client_max_body_size 10M;
  keepalive_timeout 5;

  # Static files directo por nginx
  location ~ ^/(assets|packs|images|favicon.ico|robots.txt) {
    expires max;
    add_header Cache-Control public;
    try_files $uri =404;
  }

  # App (Puma)
  location / {
    proxy_pass http://localhost:3006;

    proxy_http_version 1.1;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;

    # WebSockets / ActionCable (por si lo usas)
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }

  error_page 500 502 503 504 /500.html;
}
```