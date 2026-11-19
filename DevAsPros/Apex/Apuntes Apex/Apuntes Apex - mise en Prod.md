# Apuntes Apex - mise en Prod

## Cash Flow

En `scripts/003_after_deploy.sh`:
```bash
export PATH="/home/ubuntu/.local/share/mise/shims:$PATH"
```

En `scripts/cashflow.jobs.service`:
```bash
ExecStart=/home/ubuntu/.local/bin/mise exec -- bundle exec bin/jobs start --mode async
```

En `scripts/cashflow.puma.service`:
```bash
ExecStart=/home/ubuntu/.local/bin/mise exec -- bundle exec puma -C config/puma.rb
```

### bundler 4.1.0

Tuve que asegurarme de instalar bundler 4.1.0.

En la carpeta `~/cashflow/app`
```bash
mise exec -- gem install bundler -v 4.1.0
```

### Puma y OpenSSL 1

El despliegue completó con éxito pero al abrir la página daba 502 bad gateway. Es por Puma estar compilado con el OpenSSL de Ubuntu 20.04.

```bash
find ~/cashflow/deployments/api-gems/bundle/ruby/3.4.0 -name '*.so' -exec sh -c 'ldd "$1" | grep -E "libssl|libcrypto" | sed "s|^|$1: |"' _ {} \;
```

Salida:
```bash
.../puma-7.2.1/puma/puma_http11.so: libssl.so.1.1 => /lib/x86_64-linux-gnu/libssl.so.1.1
(solo puma enlaza el OpenSSL 1.1 del sistema)
```

En cambio el Ruby de mise usa OpenSSL 3
```bash
mise exec -- ruby -ropenssl -e 'OpenSSL::PKey::RSA.new(2048); puts OpenSSL::OPENSSL_VERSION, OpenSSL::OPENSSL_LIBRARY_VERSION'
# OpenSSL 3.5.5 27 Jan 2026   ← el Ruby de mise solo funciona bien
```

#### Corregir Puma

Primero `cd ~/cashflow/app` y luego:
```bash
PUMA_DISABLE_SSL=1 mise exec -- bundle pristine puma
```

Verificaciones:
```bash
find ~/cashflow/deployments/api-gems/bundle/ruby/3.4.0 -path '*puma-7.2.1*' -name '*.so' -exec sh -c 'ldd "$1" | grep -E "libssl|libcrypto"' _ {} \;
# (vacío)

mise exec -- bundle exec ruby -e 'require "puma"; p Puma.ssl?'
# false
```

## Eliminando chruby y ruby-install

> [!Important]
> Primero borra cualquier entrada de `~/.bashrc`, `~/.bash_profile` y/o `~/.profile`.

Se borra así en Linux:
```bash
sudo rm -rf /usr/local/share/chruby \
  /usr/local/bin/ruby-install \
  /usr/local/share/ruby-install
```

### Confirmación que mise es el que manda

Corre los comandos:
- `mise doctor`
- `ruby -v`
- `which ruby`

Deben dar salidas satisfactorias apuntando a mise.