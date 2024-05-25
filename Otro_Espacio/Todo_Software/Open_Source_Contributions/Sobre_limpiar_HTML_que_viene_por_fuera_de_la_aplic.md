# Sobre limpiar HTML que viene por fuera de la aplicación y mostrarlo.
Rails provee varias formas de hacer que un HTML externo sea válido para nuestra aplicación.
Se puede lograr/validar/permitir/verificar un HTML externo con los métodos:

- `[sanitize](https://apidock.com/rails/ActionView/Helpers/SanitizeHelper/sanitize)` [de](https://apidock.com/rails/ActionView/Helpers/SanitizeHelper/sanitize) `[ActionView::Helpers::SanitizeHelper](https://apidock.com/rails/ActionView/Helpers/SanitizeHelper/sanitize)`
- `[html_safe](https://apidock.com/rails/String/html_safe)` [de](https://apidock.com/rails/String/html_safe) `[String](https://apidock.com/rails/String/html_safe)`
- `[raw](https://apidock.com/rails/v5.2.3/ActionView/Helpers/OutputSafetyHelper/raw)` [de](https://apidock.com/rails/v5.2.3/ActionView/Helpers/OutputSafetyHelper/raw) `[ActionView::Helpers::OutputSafetyHelper](https://apidock.com/rails/v5.2.3/ActionView/Helpers/OutputSafetyHelper/raw)`
- `[safe_join](https://apidock.com/rails/ActionView/Helpers/OutputSafetyHelper/safe_join)` [de](https://apidock.com/rails/ActionView/Helpers/OutputSafetyHelper/safe_join) `[ActionView::Helpers::OutputSafetyHelper](https://apidock.com/rails/ActionView/Helpers/OutputSafetyHelper/safe_join)`

Sin embargo, al usar Rubocop y Rubocop Rails, hay un Cop que se queja sobre el uso de `raw`, `html_safe` y `safe_join`:

    Inspecting 1 file
    C
    
    Offenses:
    
    app/services/reddit_json_from_url_service.rb:37:26: C: Rails/OutputSafety: Tagging a string as html safe may be a security risk.
        safe_join([truncated.html_safe])

Al final lo resolví usando solamente `sanitize`.

**Enlaces Relacionados**

- En Betterment [escriben al respecto](https://www.betterment.com/resources/safe-html-rendering/) para entender los motivos de porqué el Cop funciona así... aunque no dan muchas ideas de qué hacer al respecto.
- [Acá está el PR](https://github.com/rubocop-hq/rubocop/pull/4320) sobre la actualización de cómo funciona dicho Cop.
- [Documentación del Cop](https://www.rubydoc.info/gems/rubocop-rails/RuboCop/Cop/Rails/OutputSafety)

**Puntos relacionados al encodeo/decodeo de HTML externo**

- [HTMLEntities](https://github.com/threedaymonk/htmlentities)
- [How do I encode/decode HTML entities in Ruby](https://stackoverflow.com/questions/1600526/how-do-i-encode-decode-html-entities-in-ruby)

