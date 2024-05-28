# Usando Tmux

## Manual Oficial
- Instalación [https://github.com/tmux/tmux/wiki/Installing#installing-tmux](https://github.com/tmux/tmux/wiki/Installing#installing-tmux)
    - Instalé en macos con `brew install tmux`
- Dividir ventanas https://github.com/tmux/tmux/wiki/Getting-Started#splitting-the-window
- Prefijo https://github.com/tmux/tmux/wiki/Getting-Started#the-prefix-key
- Crear nuevas ventana https://github.com/tmux/tmux/wiki/Getting-Started#creating-new-windows

Para crear una nueva sesión nombrada:

    tmux new -s cashflow

Cuando se arranca tmux, los comandos son ejecutados normalmente. Para llamar la atención de tmux, hay que presionar el prefijo:

    ctrl + B

## Soltando y uniendo sesiones

Con el comando

    ctrl + B y D

se “detach” la sesión y queda corriendo en segundo plano.

Con el comando:

    tmux attach -t cashflow

se vuelve a unir a la sesión previamente soltada

Listar sesiones

    $ tmux ls
    cashflow: 1 windows (created Thu Mar  7 20:13:57 2024) (attached)


## Nueva ventana

    ctrl + B y C

crea una nueva ventana. en una sesión unida.

Así luce el listar sesiones:

    $ tmux ls
    cashflow: 2 windows (created Thu Mar  7 20:13:57 2024) (attached)


## Dividir ventanas

Al dividir una ventana se crea un panel.

    C-b %

Divide la ventana en dos paneles. Uno al lado del otro.

![división horizontal pone una junto a la otra](https://paper-attachments.dropboxusercontent.com/s_06C6E8229A7B5654990A5B2569FF5ADE75430F0FF49E6E207E95E3875472E57F_1709938513458_horizontal.png)



    C-b "

Divide la ventana en dos panales. Uno debajo del otro.

![división vertical poner una debajo de la otra](https://paper-attachments.dropboxusercontent.com/s_06C6E8229A7B5654990A5B2569FF5ADE75430F0FF49E6E207E95E3875472E57F_1709938521017_vertical.png)


## Moverse entre Ventanas

    ctrl + b y 0-9

Donde 0 a 9 es el número de la ventana que se muestra en la barra de estado:

![](https://paper-attachments.dropboxusercontent.com/s_06C6E8229A7B5654990A5B2569FF5ADE75430F0FF49E6E207E95E3875472E57F_1709938387526_imagen.png)

También se puede usar:

- ctrl + B y p: previous
- ctrl + B y n: next

## Moverse entre Panales

Los paneles son la división de una ventana.

    C-b Up, C-b Down, C-b Left and C-b Right

para moverse entre paneles. Los comandos son circulares.

Con el comando

    C-b q

se muestra un número grande en cada panel. Al presionar el número impreso antes que desaparezca hace que se cambie a ese panel.

Con el comando

    C-b x

se cierra el panel actual

# Actualizando tmux mediante Homebrew en Macos

Resulta que tuve que ejecutar una actualización con Homebrew y actualizó tmux de paso. Tenía una sesión abierta y cuando quise volver a ejecutar tmux en otra terminal pasaba algo como:

```
$ tmux
[exited]
```

Ese error pasa porque haber tenido la sesión abierta. La solución fue cerrar la sesión activa y se pudo volver a lanzar tmux.

Visto en [Unix & Linux](https://unix.stackexchange.com/a/458367/47620).

# Scroll y copiar texto

En tmux el scroll normal de la terminal no va a funcionar. Hay que ejecutar una combinación de teclas.

Con esta combinación se pasa a modo navegación.

```
C-B [
```

En este modo se puede hacer scroll vertical y copiar texto.

> [!info] Copiar texto en modo navegación
> 
> Si se tienen dos paneles de modo horizontal (uno al lado del otro) al copiar texto la selección toma el contenido de ambos lados.