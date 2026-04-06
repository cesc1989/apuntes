# Para la versión 1.16 de Puntapie

## Subir versiones

- Rails: subir a 7.2.3.1
- Devise: subir a 5.0.3

## Aplicar cambios de registrations path

Los últimos cambios hechos en enlacito para completar el registro.

Este pull request: https://github.com/cesc1989/enlacito/pull/14

Incluye:
- Registro activado
- Parámetros adicionales habilitados
- Controlador de devise generado
- Mínimo de contraseña desde 10

## Montar landing básica

Tomando los cambios más recientes que haya en enlacito.

Consideraciones de la landing básica:

- Navbar con logotipo en la izquierda y botones en la derecha
- Hero simple
- Tres bloques de características
- Bloque de CTA
- Footer con ToS, Hecho en Barranquilla y el copy + año

## Aplicar cambios de confirmación de correo 📨

Todo lo que se hizo para Enlacito.

- Migración y otros: https://github.com/cesc1989/enlacito/commit/709e5200f0b62a93eceeccf9e20355b331aaa303
- Ajustes en environments: https://github.com/cesc1989/enlacito/commit/8fc7dac6006ad9709d8a850e63a84deab25361e4
- Ajustes para las pruebas:
	- Factories: https://github.com/cesc1989/enlacito/commit/ef1854fac4a94e59a4f301bdd334defe33044a15
	- envs: https://github.com/cesc1989/enlacito/commit/69c5c5d2144ca961b8029ccfbb9796c127d3f7f1
- Configuración de Resend: https://github.com/cesc1989/enlacito/commit/09810c855a1def352b34bba03a986bfdcab7b047
- Ajustes en Devise: https://github.com/cesc1989/enlacito/commit/d906b5ab275a79bab8a5beab238bf3705ab0c3a2
- Más ajustes: https://github.com/cesc1989/enlacito/commit/e367c09bbbcea4be0eece3b6ace4e7f60a5f9555
- Finalmente, correo con buenos estilos y preview para revisar: https://github.com/cesc1989/enlacito/commit/37637121b2193cd1772b7f6d03930c4c73348aa6

## Agregar todas las vistas de Devise para no joder más con esto

Ver: https://github.com/cesc1989/enlacito/commit/a555bc57e92b95350c5c466abcfbb0701fe819ce

## Corregir estilos del Sidebar

El borde del sidebar no llega hasta el fondo. La parte del navbar donde iría el logo tiene más oscuridad.

Commits:
- Quitar oscuridad a navbar brand: https://github.com/cesc1989/enlacito/commit/d44f2e66dff1531126a6cd426d5caf9a126b7a93
- Borde del sidebar: https://github.com/cesc1989/enlacito/commit/a9e4a71de42798fc6e22e435b918a036cb05c808

Resumido en Super Menú: https://github.com/devaspros/supermenu/commit/4edb657204236b5387d1632267e1ae6b08bea6c3