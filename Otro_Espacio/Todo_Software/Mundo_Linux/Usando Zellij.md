# Probando Zellij

Página web: https://zellij.dev/

Documentación: https://zellij.dev/documentation/overview

> Zellij is a workspace aimed at developers, ops-oriented people and anyone who loves the terminal. At its core, it is a terminal multiplexer (similar to tmux and screen), but this is merely its infrastructure layer.

## Paneles

Crear un panel es sencillo. Una vez lanzado zellij, presionamos CTRL + P -> N
![[zellij_panes.png]]

## Layouts

Docs de layouts: https://zellij.dev/documentation/layouts

Se puede arrancar una sesión de Zellij con varias configuraciones de inicio al crear un archivo de layouts.

En mi caso lo hice de esta forma ya que normalmente busco dos paneles verticales.

Cree carpeta `layouts` en carpeta de configuraciones de zellij
```bash
mkdir ~/.config/zellij/layouts
```

Luego cree un archivo layout básico:
```bash
touch ~/.config/zellij/layouts/two_panes.kdl
```

Y le di esta configuración:
```
layout {
    pane split_direction="vertical" {
        pane
        pane
    }
}
```

Para cargar ese archivo al iniciar una nueva sesión de Zellij ejecuto el comando así:
```bash
zellij --layout two_panes
```

Maravilloso.

## Scroll vertical y copiado

Esto sí me gusta de Zellij. No hay necesidad de activar ningún modo de navegación como en Tmux. Se puede hacer scroll vertical de manera normal y para copiar texto solo necesito seleccionar y lo copia de manera automática.

Pero hay que configurar algo para poder usar lo que haya en el porta papeles.

La configuración está en este FAQ.

Hay que primero generar el archivo de configuración de Zellij
```bash
mkdir ~/.config/zellij
zellij setup --dump-config > ~/.config/zellij/config.kdl
```

Y luego cambiar la opción `copy_command`:
```
copy_command "pbcopy"
```

Esto es para macos. No sé aún para Linux Mint.

## Aliases

Zellij parace no tomar la configuración de aliases que hice para Bash.

Parece que esto ocurre porque Zellij no está haciendo `source` de mis configuraciones al arrancar.

Y no lo hace porque tengo los dotfiles configurados en `.profile` y ese solo funciona cuando el usuario de sistema arranca una terminal. Para que pase a nivel de otros programas debo cargarlos también en el `.bashrc`:

```bash
# ~/.bashrc

if [ -f "$HOME/.bash_profile" ]; then
  . "$HOME/.bash_profile"
fi
```

Así hice luego de ver el comentario en este [issue](https://github.com/zellij-org/zellij/issues/2050).

# Enlaces

Aquí pongo algunos enlaces relacionados pero que no competen con la configuración y prueba inicial:

- Artículo en Dev donde puedo ver un alias para arrancar Zellij https://dev.to/pietrangelo/helix-and-zellij-34km
- Página de screencastas oficiales https://zellij.dev/screencasts/

# Otras cosas

Me pasó que los paneles de un tab estaban sincronizados. Al escribir en uno se reflejaba lo mismo en todos.

Encontré que Zellij tiene una [opción](https://github.com/zellij-org/zellij/discussions/1271) para eso a través del [atajo](https://github.com/zellij-org/zellij/blob/main/zellij-utils/assets/config/default.kdl#L56) de teclado:
```
CTRL + T -> S
```

![[Pasted image 20240702213153.png]]

Al hacer la misma combinación se desactiva.

## Crear nuevo panel hacia la derecha o hacía abajo

Por defecto, el panel se crea en modo horizontal. Para crearlo vertical hay que indicar la dirección usando la letra R. Con D se crea hacía abajo:

```
CTRL & P + R => a la derecha

CTRL & P + D => hacia abajo
```