# Asegurando el VPS Linux

Etiquetas: #despliegue_vps 

Todas las cosas que hay que hacer por seguridad básica.

## Recursos

- Configuración que hice para CashFlow [[Apuntes_Ciclo_02_-_Configuración_-_Cash_Flow]]
- Branko en twitter: https://x.com/brankopetric00/status/2017283246254436501
- Secure a compute instance - Linode: https://techdocs.akamai.com/cloud-computing/docs/set-up-and-secure-a-compute-instance
- Using Fail2ban to Secure Your Server: https://www.linode.com/docs/guides/using-fail2ban-to-secure-your-server-a-tutorial/

# Hardening en Linux

## Actualizar el VPS 🟢

```bash
sudo apt update && sudo apt upgrade -y
```

## Configura unattended-upgrades 🟡

Ver [[Unattended Upgrades]]

## Agregar usuario no-root 🟢

Hay que agregar un usuario de sistema para que este interactue con el VPS. Luego hay que quitar el acceso root al VPS.

```bash
useradd -m -p $(openssl passwd -1 Superman123.) -s /bin/bash -G sudo ubuntu
```

## Configura acceso por ssh para el usuario no-root 🟢

Para ingresar sin usar la contraseña que es más insegura.

Estas son las llaves públicas de la clave para ingresar como el usuario ubuntu del VPS y la que permite hacer pull del repo en GitHub.
```bash
cd /home/ubuntu

mkdir -p ~/.ssh && chmod -R 700 ~/.ssh

echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOsP7/b9+0xb4uVGvV7LqahhZmWFtuAf3xIpdF6HLMcq frajaquico@aol.com Linode" >> ~/.ssh/authorized_keys

echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDhsvMg6pUEsWzYBEwTb89bKqGCU1sQlVB74xvjsCICzHNYGkYqWieCeFURRRqz2MruJHaCCsoB3SLfEQ1X+iMAipMPsTWVK9nq46gxAStewatMLtX1rfgK81LSTbywQ4dKWBQS4B824nnPEu82AW21+V4W3IFUuodc3uqmmEzp+2AjXE70GZwvCVxyz2q4UIjzfY9SV/ald6o5ZE6oChA97ZBRJQ06q2zA1fC78+Ob3JRCKDLRgkcr3kQ0kLQVF81yG1DiqpeaCIcK4lsg97uNixgrvdFuicRsNIMfSF/fsNx/nFAxTeycKSu5aQLufVQ/bPFe2iTMIia736ZYDaqxpVQBrF2EVUlf9LGLdkX9NB64R85aOwFSSIAZxUd3zeCBq1jJBWJRMycGMWT1cBXs5uyiNTS04LimWVv7L/8apDOy9l9jsTu6uY4nKtVWq0eREKGAD5EkrStcshwatPQOStw4o+qI2D/VGtFA9v67ngxrCZ/irB5raLEsVWOYoLs= github ubuntu server" >> ~/.ssh/authorized_keys

chmod 600 ~/.ssh/authorized_keys
```

## Crea archivo `~/.ssh/config` para atajos de interacción ssh 🟢

Con esto el script de despliegue, en la parte de bajar el repo, funcionará sin problema:
```bash
cat >> ~/.ssh/config << END
Host gh
	HostName github.com
	User git
	IdentityFile ~/.ssh/gh_ubuntu
END
```

## Configura `/etc/ssh/sshd_config` 🚨🟢

> [!Info]
> Docs de sshd https://linux.die.net/man/5/sshd_config

> [!Warning]
> Mantén la sesión de terminal abierta y prueba el acceso con `ubuntu` en otra pestaña. Si algo falla, quedaría afuera del servidor.

> [!Danger]
> Este es un paso crucial. Hay bots pendientes a servidores nuevo. Es clave que esto quede bien configurado para alejar a actores maliciosos.

Aquí es donde quitamos el acceso al usuario `root`. Vamos a buscar que quede así:
```bash
Port 54321
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
PermitRootLogin no

PermitEmptyPasswords no
MaxAuthTries 3

AllowUsers ubuntu
```

Esta es una forma de lograrlo:
```bash
sudo sed -i 's/^#*Port .*/Port 54321/' /etc/ssh/sshd_config

sudo sed -i 's/^#*PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config

sudo sed -i 's/^#*PubkeyAuthentication.*/PubkeyAuthentication yes/' /etc/ssh/sshd_config

sudo sed -i 's/^#*AuthorizedKeysFile.*/AuthorizedKeysFile .ssh\/authorized_keys/' /etc/ssh/sshd_config

sed -re 's/^(PermitRootLogin)([[:space:]]+)yes/\1\2no/' -i.`date -I` /etc/ssh/sshd_config
```

### Servicio ssh para aceptar el cambio de puerto 🔑

En Ubuntu 24.04 el servicio corre con socket y no permite cambio de puerto. Debo desactivarlo y arrancar el tradicional.

Desactivar ssh.socket
```bash
systemctl stop ssh.socket
systemctl disable ssh.socket
systemctl mask ssh.socket
```

Activar ssh.service
```bash
systemctl unmask ssh.service
systemctl enable ssh.service
systemctl start ssh.service
```

### Particular de Host Hatch 🟢

En Host Hatch hay un archivo adicional que causa problema para la configuración de `PasswordAuthentication`. Se desactiva así:
```bash
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config.d/50-cloud-init.conf
```

### Reiniciar `sshd` 🟢

Finalmente podemos reiniciar el servicio `ssh`:
```bash
sudo systemctl restart sshd
```

Revisa:
```bash
sudo systemctl status sshd
```

### Revisar que no haya intentos de acceder al VPS 🔐🟢

Luego de los cambios anteriores el VPS debería recibir menos ruido de afuera. Verificar con:
```bash
sudo tail -f /var/log/auth.log
```

Debería limitarse a mis ingresos al VPS.

## Configura firewall con ufw 🟢

Sencillo:
```bash
ufw default deny incoming
ufw default allow outgoing

ufw allow ssh
ufw allow http
ufw allow https
```

Permite solo para el puerto elegido:
```bash
ufw allow 54321/tcp
ufw deny 22/tcp
```

Activa con:
```bash
ufw --force enable
```

Revisa con:
```bash
ufw status numbered
ufw status verbose
```

## Configura fail2ban 🟡

Instala
```bash
sudo apt install fail2ban
```

Configura para que quede así:
```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

[sshd]
enabled = true
port = ssh
maxretry = 3
bantime = 3600
findtime = 600
```

Reinicia y activa:
```bash
sudo systemctl restart fail2ban
sudo systemctl enable fail2ban
```