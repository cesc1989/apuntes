# Apuntes de Zed

Configuraciones y demás.

# LSP con Solargraph para Ruby

[Docs de Zed](https://zed.dev/docs/languages/ruby#language-servers)

Con esta configuración en `~/.config/zed/settings.json` se desactivan todos los LSP que Zed soporta:
```json
{
  "languages": {
    "Ruby": {
      "language_servers": []
    }
  }
}
```

Para activar solargraph hay varias formas. Opté por instalar la gema en la versión de Ruby disponible.

> [!Note]
> Como tengo varias versiones de Ruby tuve que instalar solargraph en todas esas versiones.
>
> Esta ocasión fue para ruby 3.2.5, 3.1.6 y 3.0.2

Con solargraph instalado, configuro Zed de esta forma para solo usar esta gema como LSP:
```json
{
  "languages": {
    "Ruby": {
      "language_servers": ["solargraph", "!ruby-lsp", "!rubocop"]
    }
  }
}
```

## Carga y cacheo de archivos

Hay que darle un tiempo a solargraph para que lea los archivos y gemas del proyecto donde abro el editor.

Después de eso puedo hacer `CMD + Click` en nombres de clases, métodos de instancia o macros de Rails y me lleva a la línea puntual. En algunos casos donde hay muchas coincidencias me muestra resultados de búsqueda.

> [!Note]
> El `CMD + Click` sirve si al hacer hover sobre el text aparece un  subrayado. Eso quiere decir que puedo clicarlo.


# Sincronizar Configuraciones, Extensiones y Atajos de Teclado

A la fecha, Zed, a la fecha, [no ofrece una forma para lograr esto](https://github.com/zed-industries/zed/discussions/6569). Toca copiar los archivos necesarios de manera manual.

Los archivos de interés son:

- `.config/zed/keymap.json`
- `.config/zed/settings.json`

## Sincronización de Extensiones

> [!Note]
> Según este [issue](https://github.com/zed-industries/zed/issues/19512).

Dado que Zed instala las extensiones en la carpeta `~/Library/Application\ Support/Zed/extensions/installed/` no hay forma adecuada de sincronizarlas. Lo que se puede hacer entonces es usar la llave `auto_install_extensions` en el archivo de settings.

Esta llave recibe los nombres de las extensiones. Si se les da el valor `true` se van a auto instalar cada que Zed arranque.

Esta es mi lista:
```json
  "auto_install_extensions": {
    "dockerfile": true,
    "elixir": true,
    "git-firefly": true,
    "html": true,
    "log": true,
    "make": true,
    "pylsp": true,
    "ruby": true,
    "scss": true,
    "sql": true,
    "terraform": true
  }
```

## Atajos de Teclado

Están en el archivo `.config/zed/keymap.json`.

Ahora mismo solo tengo estos dos:
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

Son para hacer un recorrido ciclico entre las pestañas de los archivos abiertos.