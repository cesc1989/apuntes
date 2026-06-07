# Usar Puma como App Server

Según gepeto sería así.

## config/puma.rb

```ruby
# frozen_string_literal: true

environment ENV.fetch("RAILS_ENV") { "production" }

# VPS 1GB → 1 worker, pocos threads
workers 1
threads 3, 3

# Bind por TCP (simple con nginx)
bind "tcp://127.0.0.1:3001"

preload_app!

# Reduce fragmentación de memoria
before_fork do
  ENV["MALLOC_ARENA_MAX"] = "2"
end

on_worker_boot do
  ActiveRecord::Base.establish_connection if defined?(ActiveRecord)
end

plugin :tmp_restart
```

## coshinotes.puma.service

```bash
~/.config/systemd/user/coshinotes.puma.service
```

Configuración:
```bash
[Unit]
Description=Coshinotes Puma
After=network.target

[Service]
Type=simple
WorkingDirectory=/home/ubuntu/coshinotes/app

Environment=RAILS_ENV=production
Environment=MALLOC_ARENA_MAX=2

ExecStart=/home/ubuntu/.rubies/ruby-3.2.5/bin/bundle exec puma -C config/puma.rb

Restart=on-failure
RestartSec=2

StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=coshinotes_puma

[Install]
WantedBy=default.target
```

Se activa así:
```bash
systemctl --user daemon-reload
systemctl --user enable coshinotes.puma
systemctl --user start coshinotes.puma
```

Comprobación:
```bash
systemctl --user status coshinotes.puma
```

## configuración de nginx

Iría así:
```nginx
# Passenger extra configs for security and/or performance
passenger_show_version_in_header off;

server {
  listen 80;
  server_name coshinotes.devaspros.com;

  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl;
  server_name coshinotes.devaspros.com;

  ssl_certificate /etc/letsencrypt/live/coshinotes.devaspros.com/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/coshinotes.devaspros.com/privkey.pem;

  root /home/ubuntu/coshinotes/app/public;
  access_log /home/ubuntu/coshinotes/app/log/nginx.access.log;
  error_log /home/ubuntu/coshinotes/app/log/nginx.error.log info;

  # ❌ NO Passenger para esta app
  passenger_enabled off;

  error_page 500 502 503 504 /500.html;
  client_max_body_size 10M;
  keepalive_timeout 5;

  # Rails vía Puma
  location / {
    proxy_pass http://127.0.0.1:3001;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }

  # WebSockets / Turbo Streams
  location /cable {
    proxy_pass http://127.0.0.1:3001;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_read_timeout 60s;
  }
}
```