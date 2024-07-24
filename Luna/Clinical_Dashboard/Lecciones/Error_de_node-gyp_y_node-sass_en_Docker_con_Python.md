# Error de node-gyp y node-sass en Docker con Python2

Relacionado [[Migrando_a_Ruby_3_-_Provider_Portal]]

Se dio al subir la versión de Ruby de la 2.7.1 a la 2.7.7.

En la versión 2.7.1 se usaba la imagen de circle ci:

    circleci/ruby:2.7.1-node-browsers

La versión de yarn en esa imagen era:

    yarn install v1.22.4

Supuestamente, la versión de Node que [instalaba](https://hub.docker.com/layers/circleci/ruby/2.7-node-browsers-legacy/images/sha256-75302fdab806a45a552c6d8073a1ef98d5f9978125f246c6344c447048d4b722?context=explore) era:

    16.13.1

En la versión 2.7.7 se usa la [imagen de circle ci](https://circleci.com/developer/images/image/cimg/ruby):

    cimg/ruby:2.7.7-browsers

Que instala:

    node 18.12.1
    yarn 1.22.18

Y en el [código fuente de la imagen](https://github.com/CircleCI-Public/cimg-ruby/blob/main/2.7/browsers/Dockerfile) se ve que no instala Python.

El error es:

    yarn install v1.22.18
    [1/5] Validating package.json...
    [2/5] Resolving packages...
    [3/5] Fetching packages...
    warning @sandipmo/find-max@1.0.4: The engine "nodejs" appears to be invalid.
    [4/5] Linking dependencies...
    warning " > webpack-dev-server@3.11.3" has unmet peer dependency "webpack@^4.0.0 || ^5.0.0".
    warning "webpack-dev-server > webpack-dev-middleware@3.7.3" has unmet peer dependency "webpack@^4.0.0 || ^5.0.0".
    [5/5] Building fresh packages...
    error /__w/physician-portal/physician-portal/node_modules/node-sass: Command failed.
    Exit code: 1
    Command: node scripts/build.js
    Arguments: 
    Directory: /__w/physician-portal/physician-portal/node_modules/node-sass
    Output:
    Building: /usr/local/bin/node /__w/physician-portal/physician-portal/node_modules/node-gyp/bin/node-gyp.js rebuild --verbose --libsass_ext= --libsass_cflags= --libsass_ldflags= --libsass_library=
    gyp info it worked if it ends with ok
    gyp verb cli [
    gyp verb cli   '/usr/local/bin/node',
    gyp verb cli   '/__w/physician-portal/physician-portal/node_modules/node-gyp/bin/node-gyp.js',
    gyp verb cli   'rebuild',
    gyp verb cli   '--verbose',
    gyp verb cli   '--libsass_ext=',
    gyp verb cli   '--libsass_cflags=',
    gyp verb cli   '--libsass_ldflags=',
    gyp verb cli   '--libsass_library='
    gyp verb cli ]
    gyp info using node-gyp@3.8.0
    gyp info using node@18.12.1 | linux | x64
    gyp verb command rebuild []
    gyp verb command clean []
    gyp verb clean removing "build" directory
    gyp verb command configure []
    gyp verb check python checking for Python executable "python2" in the PATH
    gyp verb `which` failed Error: not found: python2
    # (..)
    gyp ERR! stack Error: Can't find Python executable "python", you can set the PYTHON env variable.
    gyp ERR! node -v v18.12.1
    gyp ERR! node-gyp -v v3.8.0

Intenté usando una [imagen legacy](https://hub.docker.com/r/circleci/ruby/tags?page=1&name=2.7) de circle ci. Esta:

    circleci/ruby:2.7-node-browsers-legacy

pero esa instala Ruby 2.7.5 y falla el proceso.

## Correspondencia de versiones entre node y node-sass

Tabla completa de [versiones](https://github.com/sass/node-sass#node-version-support-policy) de node soportadas por node-sass.

Node anterior:

    16.13.1

Node actual:

    18.12.1


## ¿Quién necesita node-sass?

Pues webpacker:

    # yarn.lock
    "@rails/webpacker@4.2.2":
      version "4.2.2"
      node-sass "^4.13.0"
    
    node-sass@^4.13.0:
      version "4.14.1"
      node-gyp "^3.8.0"
    
    node-gyp@^3.8.0:
      version "3.8.0"

Probamos instalando Python con esta la action oficial [setup-python](https://github.com/actions/setup-python). Acá mencionan las [variables de entorno](https://github.com/actions/setup-python/blob/main/docs/advanced-usage.md#environment-variables) disponibles.


    pythonLocation: /__w/_tool/Python/2.7.18/x64
    PKG_CONFIG_PATH: /__w/_tool/Python/2.7.18/x64/lib/pkgconfig
    Python_ROOT_DIR: /__w/_tool/Python/2.7.18/x64
    Python2_ROOT_DIR: /__w/_tool/Python/2.7.18/x64
    Python3_ROOT_DIR: /__w/_tool/Python/2.7.18/x64
    LD_LIBRARY_PATH: /__w/_tool/Python/2.7.18/x64/lib

Y con esto se soluciona el problema de Python pero ahora sale otro error.

    gyp verb check python checking for Python executable "python2" in the PATH
    gyp verb `which` succeeded python2 /__w/_tool/Python/2.7.18/x64/bin/python2
    gyp verb check python version `/__w/_tool/Python/2.7.18/x64/bin/python2 -c "import sys; print "2.7.18
    gyp verb check python version .%s.%s" % sys.version_info[:3];"` returned: %j


## El Nuevo Error error: ‘remove_cv_t’ is not a member of ‘std’; did you mean ‘remove_cv’?

Ya lo habíamos visto en la migración [+Migrando a Ruby 3](https://paper.dropbox.com/doc/Migrando-a-Ruby-3-Mx9V178363eWJuQa4z0Bw) 

    /github/home/.node-gyp/18.12.1/include/node/v8-internal.h:646:38: error: ‘remove_cv_t’ is not a member of ‘std’; did you mean ‘remove_cv’?
      |             !std::is_same<Data, std::remove_cv_t<T>>::value>::Perform(data);
      |                                      ^~~~~~~~~~~
      |                                      remove_cv
    /github/home/.node-gyp/18.12.1/include/node/v8-internal.h:646:38: error: ‘remove_cv_t’ is not a member of ‘std’; did you mean ‘remove_cv’?
      |             !std::is_same<Data, std::remove_cv_t<T>>::value>::Perform(data);
      |                                      ^~~~~~~~~~~
      |                                      remove_cv
    /github/home/.node-gyp/18.12.1/include/node/v8-internal.h:646:50: error: template argument 2 is invalid
      !std::is_same<Data, std::remove_cv_t<T>>::value>::Perform(data);
                                                  ^
    /github/home/.node-gyp/18.12.1/include/node/v8-internal.h:646:63: error: ‘::Perform’ has not been declared
      |             !std::is_same<Data, std::remove_cv_t<T>>::value>::Perform(data);
      |                                                               ^~~~~~~
    In file included from ../src/binding.cpp:1:

Y parece solucionarse así:

    CXXFLAGS="--std=c++17" yarn install

Pero vuelve otro error.

## Otro error:
    node:internal/crypto/hash:71
    this[kHandle] = new _Hash(algorithm, xofLen);
                    ^
    Error: error:0308010C:digital envelope routines::unsupported
      at new Hash (node:internal/crypto/hash:71:19)
      at Object.createHash (node:crypto:133:10)
      at module.exports (/__w/physician-portal/physician-portal/node_modules/webpack/lib/util/createHash.js:135:53)
      at NormalModule._initBuildHash (/__w/physician-portal/physician-portal/node_modules/webpack/lib/NormalModule.js:417:16)
      at handleParseError (/__w/physician-portal/physician-portal/node_modules/webpack/lib/NormalModule.js:471:10)
      at /__w/physician-portal/physician-portal/node_modules/webpack/lib/NormalModule.js:503:5
      at /__w/physician-portal/physician-portal/node_modules/webpack/lib/NormalModule.js:358:12
      at /__w/physician-portal/physician-portal/node_modules/loader-runner/lib/LoaderRunner.js:373:3
      at iterateNormalLoaders (/__w/physician-portal/physician-portal/node_modules/loader-runner/lib/LoaderRunner.js:214:10)
      at iterateNormalLoaders (/__w/physician-portal/physician-portal/node_modules/loader-runner/lib/LoaderRunner.js:221:10)
      at /__w/physician-portal/physician-portal/node_modules/loader-runner/lib/LoaderRunner.js:236:3
      at context.callback (/__w/physician-portal/physician-portal/node_modules/loader-runner/lib/LoaderRunner.js:111:13)
      at /__w/physician-portal/physician-portal/node_modules/babel-loader/lib/index.js:59:71 {
    opensslErrorStack: [ 'error:03000086:digital envelope routines::initialization error' ],
    library: 'digital envelope routines',
    reason: 'unsupported',
    code: 'ERR_OSSL_EVP_UNSUPPORTED'

Intenté agregando esta variable:

    NODE_OPTIONS=--openssl-legacy-provider

pero no se puede en la sección de envs del workflow:

    Post job cleanup.
    /usr/bin/docker exec  477ee8f6dcc8f7fbcac07a8121e858aa89e07f0906058213fc6d44bfd9305dc4 sh -c "cat /etc/*release | grep ^ID"
    /__e/node16/bin/node: --openssl-legacy-provider is not allowed in NODE_OPTIONS

Sobre este error:

- [Stack Overflow 1](https://stackoverflow.com/questions/74548318/how-to-resolve-error-error0308010cdigital-envelope-routinesunsupported-no)
- [Stack Overflow 2](https://stackoverflow.com/questions/69692842/error-message-error0308010cdigital-envelope-routinesunsupported)

Voy a intentar cambiar la configuración. Ya fue suficiente.

