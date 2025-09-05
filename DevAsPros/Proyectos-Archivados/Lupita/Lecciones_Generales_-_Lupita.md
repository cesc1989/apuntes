# Lecciones Generales - Lupita

## Desinstalar e Instalar Correctamente VirtualBox en macOS Mojave

En la más reciente versión de OSX hay un problema para instalar VirtualBox. Hay unos permisos que deben otorgarse y solo se puede hacer al intentar instalarlo y no antes.

La forma correcta de instalar se describe en [este artículo de OSXDaily](http://osxdaily.com/2018/12/31/install-run-virtualbox-macos-install-kernel-fails/). La forma es yendo a las configuraciones de *Seguridad y Privacidad*, apartado *General* y ahí dar permisos a “Oracle America, Inc”.

Luego, volver a iniciar al proceso de instalación y ahora el proceso sí debería completarse como es debido.

Ahora, [para desinstalar](http://osxdaily.com/2019/01/25/uninstall-virtualbox-mac-completely/), la mejor forma es utilizando el mismo *.dmg* de la instalación. Al abrirlo, hay un archivo que se llama *VirtualBox_Uninstall.tool*  el cual es un *script* de bash que:


- Pedirá permisos: insertar clave
- Preguntará si desinstalar: Yes/No

Y hará una desinstalación completa de VirtualBox en el mac.

