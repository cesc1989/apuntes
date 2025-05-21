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

