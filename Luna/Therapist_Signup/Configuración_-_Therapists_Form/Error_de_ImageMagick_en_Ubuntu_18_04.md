# Error de ImageMagick en Ubuntu 18.04

Este error apareció cuando corrí la suite de pruebas(también pasó en Patient Forms) luego de instalar Image Magick según las [instrucciones de Backendstuff](https://github.com/cesc1989/backendstuff/blob/master/dependencies/install_image_magick.sh).

    MiniMagick::Error:
    
    `convert -size 600x120 xc:white /vagrant/storage/canvas20200113-32333-1c5zpxe.png` failed with error:
    
    convert: no encode delegate for this image format `XC' @ error/constitute.c/WriteImage/1226.

En el foro de Image Magick encuentro que puede que la compilación no tenga todos los binarios necesarios. Se puede [verificar viendo la lista DELEGATES de la versión instalada](https://imagemagick.org/discourse-server/viewtopic.php?t=35098):

    vagrant@vagrant:/vagrant$ convert --version
    Version: ImageMagick 7.0.9-16 Q16 x86_64 2020-01-13 https://imagemagick.org
    Copyright: © 1999-2020 ImageMagick Studio LLC
    License: https://imagemagick.org/script/license.php
    Features: Cipher DPC HDRI OpenMP(4.5) 
    Delegates (built-in): xml zlib

Así se veía en el MBP de Ideaware:

    ✘ fquintero@Francisco-Quinteros-MacBook-Pro ~ ❯ convert --version
    Version: ImageMagick 6.9.10-66 Q16 x86_64 2019-09-23 https://imagemagick.org
    Copyright: © 1999-2019 ImageMagick Studio LLC
    License: https://imagemagick.org/script/license.php
    Features: Cipher DPC Modules 
    Delegates (built-in): bzlib freetype jng jp2 jpeg lcms ltdl lzma png tiff webp xml zlib

Corregí [instalando estas otras dependencias](https://mindaslab.github.io/2018/11/22/install-latest-version-of-imagemagick-in-ubuntu-18-04.html):

    sudo apt-get install \
      build-essential \
      checkinstall \
      libx11-dev \
      libxext-dev \
      zlib1g-dev \
      libpng-dev \
      libjpeg-dev \
      libfreetype6-dev \
      libxml2-dev

Recompile ImageMagick con los siguientes comandos(desde la carpeta donde se descargó):

    sudo ./configure &&
    sudo make && 
    sudo make install &&
    sudo ldconfig /usr/local/lib

Una vez corregido, luce así en Vagrant:

    convert --version
    Version: ImageMagick 7.0.9-16 Q16 x86_64 2020-01-13 https://imagemagick.org
    Copyright: © 1999-2020 ImageMagick Studio LLC
    License: https://imagemagick.org/script/license.php
    Features: Cipher DPC HDRI OpenMP(4.5) 
    Delegates (built-in): freetype jng jpeg png x xml zlib

Y las pruebas corren con normalidad.

## Solución Ideal

- Instalar desde los repositorios oficiales del sistema operativo: [Ask Ubuntu](https://askubuntu.com/a/1176071)

