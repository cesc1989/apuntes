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

# Error objc69365: +NSNumber initialize al consultar en rails console

Cuando hacía alguna consulta en la consola de Rails me daba este error:

```
objc[69365]: +[NSNumber initialize] may have been in progress in another thread when fork() was called.
objc[69365]: +[NSNumber initialize] may have been in progress in another thread when fork() was called. We cannot safely call it or ignore it in the fork() child process. Crashing instead. Set a breakpoint on objc_initializeAfterForkError to debug.
```

y me sacaba de la consola.

La solución que encontré de momento es desactivar spring mediante variable:

```
export DISABLE_SPRING=true

rcon

[1] pry(main)> Account.first
  Account Load (814.9ms)  SELECT "accounts".* FROM "accounts" ORDER BY "accounts"."id" ASC LIMIT $1  [["LIMIT", 1]]
```

Visto en [Stack Overflow](https://stackoverflow.com/a/68910843/1407371).