# Teclados

Cualquier apunte sobre distintas configuraciones de teclados. Dado a que uso Macos y/o Linux, y algunos teclados vienen en idiomas diferentes, toca tener un lugar donde referenciar todos estos asunto.

## Teclado externo Logitech no reconoce teclas < y >

Este es un duplicado de [[Usando_una_Mac_con_Chip_M1#Teclado externo Logitech no reconoce teclas < y >]]. Duplico para no dañar el enlace que apunta a ese tema ya.

Pues resulta que no podía usar las teclas menor que y mayor que aunque usará el layout Latin America o Spanish Legacy.

También probé Karabiner pero no funcionó. Lo que sí sirvió fue usar [este repo](https://github.com/neosergio/Latam-Keyboard) y sus configuraciones.

El comando final es:

```bash
git clone https://github.com/neosergio/Latam-Keyboard.git && cd Latam-Keyboard && cp -v Latam*.* ~/Library/Keyboard\ Layouts/
```

Selecciono el idioma de “Otras” fuentes en el “Input Sources” y al fin funcionan esas teclas con el teclado externo.

![[001.teclado.latam.png]]

Con el teclado del macbook toca invertir los idiomas porque al teclear la tecla en cuestión queda invertida:
 
```
< = |
    
> = °
```

Pero mientras use el teclado externo estaremos bien con ese idioma.

Visto en [Superuser](https://superuser.com/a/1759650/372807).

## Instalar idioma US International Alt Gr Dead keys en Macos

Este es la configuración de idioma que permite usar un teclado ANSI y tener las tildes y la virgulilla para cuando necesito escribir en Español.

Mis fuentes son:

- Respuesta en Ask Different -> https://apple.stackexchange.com/a/410839
- La sugerencia es instalar este layout -> https://github.com/xv0x7c0/osx-us-altgr-intl
- Se medio por buscar más alternativas y encontré este post -> https://carlosvaz.com/posts/us-international-with-altgr-dead-keys-on-macos/
- El cual me llevó a este repo. De aquí tomé el layout en uso -> https://github.com/carjorvaz/macos-us-altgr-intl?tab=readme-ov-filed

 ### Instrucciones

Descargar el repo y descomprimir. Luego copiar el archivo de layout a la carpeta donde Macos carga los layouts:
```bash
cp ~/Downloads/macos-us-altgr-intl-master/us-altgr-intl.keylayout ~/Library/Keyboard\ Layouts/
```

Si no aparece en el espacio de configuraciones de layouts en los settings de teclado, reinicia el computador.

![[002.teclado.alt.gr.dead.keys.png]]