# Install WKHTMLTOPDF in Edge in Alpine 3.14

To setup the Docker file to install this package I did this:
```bash
RUN set -ex \
    && apk add --no-cache \
      bash \
      (...)
      wkhtmltopdf \
```

However, when I tried to run the command:
```
wkhtmltopdf http://google.com google.pdf
```

It does not complete successfully but says something about a Segmentation Fault:
```
/usr/src/app # wkhtmltopdf --version
wkhtmltopdf 0.12.6
/usr/src/app # wkhtmltopdf --enable-local-file-access http://google.com google.pdf
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-root'
Loading page (1/2)
Segmentation fault (core dumped)                             ] 30%
```

> [!Info]
> I tried the flag `--enable-local-file-access` because the [first result](https://stackoverflow.com/a/62315247/1407371) suggested that but it was for a different error.

## Install Fonts

This is the Dockerfile in Patient Self Report:
```bash
RUNTIME_PACKAGES="(...) wkhtmltopdf libxrender libxext gcompat fontconfig freetype ttf-dejavu ttf-droid ttf-freefont ttf-liberation"

RUN set -ex \
    && apk add --upgrade --no-cache --virtual .app-builddeps \
    $BUILD_PACKAGES \
    $DEV_PACKAGES
```

So I'm trying now installing fonts as suggested here: https://github.com/wkhtmltopdf/wkhtmltopdf/issues/4603#issuecomment-598471663