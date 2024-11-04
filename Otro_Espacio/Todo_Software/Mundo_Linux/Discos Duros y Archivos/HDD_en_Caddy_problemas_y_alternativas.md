# HDD en Caddy: problemas y alternativas
Este documento es una continuación de [[Particionando_en_SSD_y_HDD_para_archivos_para_Linu]]

# Contexto

Al PC Lenovo le compré una unidad SSD de 240GB. También compré un caddy para poner el antiguo HDD de 1TB en el caddy y quitar la unidad óptica. De esta forma el equipo quedaría con esta configuración:

- Disco principal: SSD 240GB
- Disco secundario: HDD 1TB (mediante caddy en unidad óptica)

# Problemas

Una vez completa la instalación del SSD como disco principal, ajustar el HDD en el Caddy y ponerlo en el lugar respectivo, me di a la tarea de instalar Linux Mint 21.3 en el SSD.

Lo primero que noté fue que el HDD no aparecía en el programa de Instalación de Linux desde la unidad booteable.

Luego, cuando el sistema estaba instalado, el HDD seguía sin aparecer. Esto significaba problemas.

El HDD no aparecía en sistema al usar comandos como:

- `lsblk`
- `fdisk -l`
- `lshw`

También me di cuenta que aparecía este elemento en la lista de boot: *ATAPI CD0*. ChatGPT explica que eso podría ser la unidad óptica así que quité el caddy y prendí el PC y aún salía. Por lo que eso no era un camino a la solución.

Al buscar y preguntar a ChatGPT identifiqué estas posibles causas:

- Caddy no funcional
- Mala configuración de la BIOS

## Caddy no funcional

Esta es la causa más común. Si la unidad óptica funcionaba y el disco funcionaba en su antigua posición, es más probable que el caddy esté defectuoso.

Intenté sacando el caddy y reajustando el disco. Me di cuenta también que el caddy no tenía el seguro que se debe atornillar a la placa. Lo que hice fue quitarle el seguro que tenía la unidad óptica y se lo atornillé al caddy. En todo caso seguía sin ser reconocido por la BIOS.

También probé limpiar los contactos SATA del caddy pero sigue sin ser reconocido. Solo me queda probar lo último que será mediante cable.

**18 de Mayo**
Probé conectar el disco mediante cable SATA-USB y funcionó. Pude formatearlo y crear una nueva partición. El daño será el Caddy.

**¿Qué queda por probar?**

- Conectar el disco mediante cubierta SATA-USB

## Mala configuración de la BIOS

Aquí probé activando y desactivando varias cosas:

- Desactivé Intel Virtualization
- SATA: cambié AHCI a Compatibility

¿**Qué queda por probar?**

- reiniciar la configuración de la BIOS
- desactivar la unidad óptica
- desactivar el anti-theft

