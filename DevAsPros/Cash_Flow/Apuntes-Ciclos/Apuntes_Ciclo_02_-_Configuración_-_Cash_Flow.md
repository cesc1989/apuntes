# Apuntes Ciclo 02 - Configuración - Cash Flow

# Permisos en Linode

El token debe tener permisos para:

- Linodes
- Images
- Events

# Sobre SQLite en producción

En este [hilo en Reddit](https://www.reddit.com/r/rails/comments/k4vlqo/is_anyone_using_sqlite_on_production_either_side/) comparten algunos comentarios positivos y enlaces sobre productos que usan SQLite como base de datos en prod.

[Otro hilo](https://www.reddit.com/r/rails/comments/y46our/sharing_my_exp_so_far_with_using_sqlite_in/). Incluso menciona como copiar el archivo .sql
```bash
scp my-remote-server:/db/production.sqlite3 ./db/development.sqlite3
```

La cosa luce bastante bien y simple así que sigo convencido de esta prueba.

Rails tirará una advertencia si se usa SQLite3 en producción. [El PR explica](https://github.com/rails/rails/pull/42191) que se puede silenciar.

# Crear imagen con dependencias

## Seguridad

- Cambiar configuración de sshd_config → [https://superuser.com/a/759505/372807](https://superuser.com/a/759505/372807)
- UFW → [https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-20-04)
- Activar ufw sin intervención del usuario [https://serverfault.com/questions/790143/ufw-enable-requires-y-prompt-how-to-automate-with-bash-script](https://serverfault.com/questions/790143/ufw-enable-requires-y-prompt-how-to-automate-with-bash-script)


## Correr comando desde root en nombre de otro usuario

Explicación  [https://askubuntu.com/a/978467/167553](https://askubuntu.com/a/978467/167553)
```bash
sudo -u USERNAME bash -c 'whoami;echo $USER'

# ejemplo
sudo -u ubuntu bash -c 'mkdir folder_name'
```

## Una línea para agregar usuario de sistema

 Single liner to create a sudo user with home directory and password.: https://stackoverflow.com/a/44363818/1407371
 
```bash
useradd -m -p $(openssl passwd -1 ${PASSWORD}) -s /bin/bash -G sudo ${USERNAME}
```

# Llaves Públicas

## Para acceso al servidor creado mediante la imagen

En el archivo Packer está esta línea:
```bash
authorized_keys = ["ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOsP7/b9+0xb4uVGvV7LqahhZmWFtuAf3xIpdF6HLMcq frajaquico@aol.com Linode"]
```

Esa es la llave pública previamente generada antes de crear la imagen.

De esa forma cuando se crea el servidor nuevo a partir de la imagen, el archivo `authorized_keys` existirá con esa línea:
```bash
ubuntu@localhost:~$ cat .ssh/authorized_keys 
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIOsP7/b9+0xb4uVGvV7LqahhZmWFtuAf3xIpdF6HLMcq frajaquico@aol.com Linode
```

Así que para acceder a un servidor nuevo ejecuto el comando ssh así:
```bash
ssh -i ~/.ssh/linode ubuntu@IP
```

donde `~/.ssh/linode` es la llave privada.

## Para acceso a GitHub. Bidireccional

Para poder hacer pull al repositorio desde el servidor hay que configurar la llave privada en el servidor.

Esto es solo subir la llave privada. Así queda al cargarla:
```bash
ubuntu@localhost:~$ ll .ssh/
-rw-rw-r--  1 ubuntu ubuntu   67 Jul 13  2023 config
-rw-------  1 ubuntu ubuntu 2610 Jul 13  2023 gh_ubuntu
```

Por otro lado…

## ¿Por qué está presente la llave pública en el servidor?

Así aparece en el archivo `.ssh/authorized_keys`
```bash
ubuntu@localhost:~$ cat .ssh/authorized_keys 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDhsvMg6pUEsWzYBEwTb89bKqGCU1sQlVB74xvjsCICzHNYGkYqWieCeFURRRqz2MruJHaCCsoB3SLfEQ1X+iMAipMPsTWVK9nq46gxAStewatMLtX1rfgK81LSTbywQ4dKWBQS4B824nnPEu82AW21+V4W3IFUuodc3uqmmEzp+2AjXE70GZwvCVxyz2q4UIjzfY9SV/ald6o5ZE6oChA97ZBRJQ06q2zA1fC78+Ob3JRCKDLRgkcr3kQ0kLQVF81yG1DiqpeaCIcK4lsg97uNixgrvdFuicRsNIMfSF/fsNx/nFAxTeycKSu5aQLufVQ/bPFe2iTMIia736ZYDaqxpVQBrF2EVUlf9LGLdkX9NB64R85aOwFSSIAZxUd3zeCBq1jJBWJRMycGMWT1cBXs5uyiNTS04LimWVv7L/8apDOy9l9jsTu6uY4nKtVWq0eREKGAD5EkrStcshwatPQOStw4o+qI2D/VGtFA9v67ngxrCZ/irB5raLEsVWOYoLs= github ubuntu server
```

**Respuesta:** Está presente al configurar el acceso desde el servidor a github. Se agrega de manera automática.


# Configuración GH

1. Subí la llave privada al servidor
2. Arranqué el agente ssh: `eval $(ssh-agent)`
3. Agregué la llave privada al agente: `ssh-add ~/.ssh/gh_ubuntu`
4. Probé la conexión a github: `ssh -T git@github.com`

