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

First, I tried with `ttf-dejavu` and got this error
```
/usr/src/app # wkhtmltopdf http://google.com google.pdf
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-root'
Loading page (1/2)
Error compiling builtin:                                     ] 30%
Fatal error compiling builtin function 'apply': Segmentation fault (core dumped)
```

## List of deps in other installations of wkhtmltopdf

For Docker image ruby-3.1.0-alpine-3.14

This is the list of deps used to install wkhtmltopdf in Patient Self Report:
```
wkhtmltopdf
libxrender
libxext
gcompat
fontconfig
freetype
ttf-dejavu
ttf-droid
ttf-freefont
ttf-liberation
```

And these are the `-dev` version deps:
```
git libxml2-dev libxslt-dev fontconfig-dev freetype-dev libx11-dev libxext-dev libxrender-dev
```

Where the wkhtmltopdf should be
```bash
fontconfig-dev # matching fontconfig
freetype-dev   # matching freetype
libx11-dev     # matching  ???
libxext-dev    # matching libxext
libxrender-dev # matching libxrender
```

For Docker image ruby:3.0.2-alpine3.14

This is the list of deps used to install wkhtmltopdf in Therapist Signup:
```
wkhtmltopdf
libxrender
libxext
gcompat
fontconfig
freetype
ttf-dejavu
ttf-droid
ttf-freefont
ttf-liberation
ttf-ubuntu-font-family
```

and this is the `-dev` list used in Therapist Signup:
```
git libxml2-dev libxslt-dev fontconfig-dev freetype-dev libx11-dev libxext-dev libxrender-dev
```

It's the same as in PSR.

## Continue Fonts installation

So, let's try installing the combined list of `-dev` and runtime deps used in Patient Self Report:
```bash
RUN set -ex \
    && apk add --no-cache \
		wkhtmltopdf \
		fontconfig-dev \
		freetype-dev \
		libx11-dev \
		libxext-dev \
		libxrender-dev \
		libxrender \
		libxext \
		gcompat \
		fontconfig \
		freetype \
		ttf-dejavu \
		ttf-droid \
		ttf-freefont \
		ttf-liberation \
```

Still returns an error:
```
/usr/src/app # wkhtmltopdf http://google.com google.pdf
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-root'
Loading page (1/2)
Error compiling builtin:                                     ] 30%
Fatal error compiling builtin function 'apply': Segmentation fault (core dumped)
/usr/src/app # exit
```

> [!important]
> this same error is returned in alpha pods for Patient Self Report and Therapist Signup.

PSR:
```
$ alpha_service_pod forms 5d869bbbbf-bz6jp
/usr/src/app # wkhtmltopdf http://google.com google.pdf
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-root'
Loading page (1/2)
Error compiling builtin:                                     ] 30%
Fatal error compiling builtin function 'apply': Segmentation fault (core dumped)
/usr/src/app # which wkhtmltopdf
/usr/bin/wkhtmltopdf
```

TS:
```
$ alpha_service_pod therapists 797d8d5fdb-42mkr
/usr/src/app # /usr/bin/wkhtmltopdf --version
wkhtmltopdf 0.12.6
/usr/src/app # /usr/bin/wkhtmltopdf http://google.com google.pdf
QStandardPaths: XDG_RUNTIME_DIR not set, defaulting to '/tmp/runtime-root'
Loading page (1/2)
Error compiling builtin:                                     ] 30%
Fatal error compiling builtin function 'apply': Segmentation fault (core dumped)
```

---

Etiquetas: #wkhtmltopdf #alpine