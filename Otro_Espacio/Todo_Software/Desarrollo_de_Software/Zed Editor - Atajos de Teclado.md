# Atajos de Teclado en Zed

Estoy probando este editor por que tiene dos cosas que me gustan mucho y en Sublime Text no se pueden:

1. Barra lateral derecha para el explorador de archivos
2. Breadcrumb automático
	1. En Sublime esto tocaría con un plugin

Pero no me convence del todo así que pondré por aquí los atajos de teclado para poder refrescar.

# En Macos

## Generales

`CMD` + `R`: Muestra/Oculta panel derecho

## Edición de Texto

`CMD` + `Shift` + `Z`: rehacer

`CMD` + `1..9`: duplica el editor hacía la derecha.

¿Cómo funciona?

- Si hay dos editores y se presiona `CMD` + `3`, abre un tercer editor.
- Si hay dos editores, `CMD` + `1` o `2` mueve el cursor entre editores.

`CMD` + `D`: duplica el cursor. Como en Sublime Text.

`CMD` + `Shift` + `K`: elimina la línea donde está el cursor.

## Modificando los Keymaps

Para poder navegar entre diferentes pestañas sin abrir el modal ese raro. Primero:

`CMD` + `k`, `CMD` + `S` para abrir el archivo `~/.config/zed/keymap.json`. En ese archivo ponemos esto:

```json
[
  {
    "context": "Pane",
    "bindings": {
      "ctrl-tab": "pane::ActivateNextItem",
      "ctrl-shift-tab": "pane::ActivatePrevItem"
    }
  }
]
```

Y ya se puede cambiar entre pestañas presionando `ctrl` + `tab` para ir a la siguiente a la derecha. Y para ir hacía la izquierda `ctrl` + `shift` + `tab`.

Visto aquí: https://github.com/zed-industries/zed/discussions/6903
