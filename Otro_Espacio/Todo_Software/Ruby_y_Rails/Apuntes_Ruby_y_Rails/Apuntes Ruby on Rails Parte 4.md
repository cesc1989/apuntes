# Error de Nokogiri al crear imagen Docker

Me topé con este error al intenter crear una imagen que usaba Ruby 3.0.2 Alpine 3.14

```
Caused by:
LoadError: Error loading shared library ld-linux-aarch64.so.1: No such file or directory (needed by /usr/local/bundle/gems/nokogiri-1.15.4-aarch64-linux/lib/nokogiri/3.0/nokogiri.so) - /usr/local/bundle/gems/nokogiri-1.15.4-aarch64-linux/lib/nokogiri/3.0/nokogiri.so
```

No sé porque explotaba eso específicamente en mi Docker local y no en el CI.

La solución la menciona la [documentación de Nokogiri](https://nokogiri.org/tutorials/installing_nokogiri.html#linux-musl-error-loading-shared-library). Hay que instalar gcompat.

Comando de Alpine:
```bash
apk add gcompat
```

Instrucción en Docker:
```bash
RUN set -ex \
    && apk update \
    && apk add build-base \
         postgresql-dev \
         tzdata \
         nodejs \
         git \
         libxml2-dev \
         libxslt-dev \
         gcompat \
         openssl \
         yarn
```