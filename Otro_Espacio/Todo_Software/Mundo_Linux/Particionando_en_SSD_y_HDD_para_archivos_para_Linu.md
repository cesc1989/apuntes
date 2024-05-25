# Particionando en SSD y HDD para archivos para Linux Mint

> **NOTA IMPORTANTE: No conectar a internet cuando se hacen instalaciones de Linux Mint.**

Así lo sugieren también [aquí](https://itsfoss.com/install-linux-mint/#step-4-install-linux-mint).

![](https://paper-attachments.dropboxusercontent.com/s_F870AE36660F15F358486BFA502CBBF3872EE695C74EC8682B3505D946D48CDD_1715720847530_imagen.png)

# Acerca de este documento

Compré una unidad SSD para instalarle Linux Mint como sistema principal y de esta forma dejar el HDD (1TB) para solo archivos (documentos, imágenes, música y películas para Plex). Normalmente, sé cómo particionar un mismo disco pero ya siendo dos diferentes me entra la duda. ¿Cómo se hace?

# Conclusión

La partición debería quedar así.

    sda disk
    
    sda1 300MB /boot/efi
    sda2 90GB / # ssd
    sda3 8GB swap # ssd
    sda4 130GB /home # ssd
    
    sdb disk
    
    sdb1 15GB # qué es este disco?
    sdb2 1TB /llamrei # pura peliculas, series y anime

Y así quedó luego de instalado el SSD y creado las particiones:

    $ lsblk
    NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
    sda      8:0    0 223,6G  0 disk 
    ├─sda1   8:1    0   285M  0 part /boot/efi
    ├─sda2   8:2    0  83,8G  0 part /
    ├─sda3   8:3    0   7,5G  0 part [SWAP]
    └─sda4   8:4    0   132G  0 part /home
    sdb      8:16   0  14,9G  0 disk 
    └─sdb1   8:17   0  14,9G  0 part 
# Estado actual de la partición

Así está particionado y al detalle:

    Disk /dev/sda: 931,53 GiB, 1000204886016 bytes, 1953525168 sectors
    Disk model: ST1000LM024 HN-M
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 4096 bytes
    I/O size (minimum/optimal): 4096 bytes / 4096 bytes
    Disklabel type: gpt
    Disk identifier: C5401CAD-9500-4B9B-A9FA-7B8235A0FC13
    
    Device         Start        End    Sectors   Size Type
    /dev/sda1       2048    2050047    2048000  1000M Windows recovery environment
    /dev/sda2    2050048    2582527     532480   260M EFI System
    /dev/sda3    2582528    2844671     262144   128M Microsoft reserved
    /dev/sda4    2844672  159094783  156250112  74,5G Linux filesystem
    /dev/sda5  159094784  647376895  488282112 232,9G Linux filesystem
    /dev/sda6  647376896 1953523711 1306146816 622,8G Microsoft basic data
    
    
    Disk /dev/sdb: 14,94 GiB, 16013942784 bytes, 31277232 sectors
    Disk model: SanDisk SSD U110
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: gpt
    Disk identifier: A3B7CBA2-2CDF-4705-ABC4-DC0A33C97286
    
    Device     Start      End  Sectors  Size Type
    /dev/sdb1   2048 31277055 31275008 14,9G unknown


## Destacados
- sda1: se puede borrar?
- sda3: se puede borrar?
- sdb1: se puede borrar?
# Última vez que particioné

Removí todas las particiones excepto:

    dev/sda1 (ntfs)
    dev/sda2 (efi)
    dev/sda3
    dev/sdb

Cree las particiones de esta forma:

    dev/sda4 (ext4) -> / 80GB
    dev/sda5 (ext4) -> /home 250GB
    dev/sda6 (fat32) -> resto del espacio

En esta ocasión, ¿cómo debería ser? ¿importará lo de EFI?

Si lo hago así:

    dev/sda4 (ext4) -> / 80GB (SSD)
    dev/sda6 (etx4) -> /home 1TB (HDD)

Pasan varias cosas:

- ¿El resto del espacio del SSD?
- ¿Hago partición para swap?
# Sugerencias encontradas

En este [post](https://superuser.com/questions/908080/linux-partitioning-with-ssd-and-hdd).

    / on SSD
    /home on HDD
    /var on HDD
    /swap on HDD


> PS: add noatime flag in your ext4 filesystems to speedup you HDD and avoid writes in your SSD.


## Fuentes
- [Installing Linux Mint on an SSD while keeping data on an HDD](https://superuser.com/questions/1217977/installing-linux-mint-on-an-ssd-while-keeping-data-on-an-hdd)
- [Linux partitioning with SSD and HDD](https://superuser.com/questions/908080/linux-partitioning-with-ssd-and-hdd)
- [How do I install linux mint on my system which has both SSD and HDD?](https://unix.stackexchange.com/questions/618848/how-do-i-install-linux-mint-on-my-system-which-has-both-ssd-and-hdd)
    - Este es donde mandar a leer “Filesystem Hierarchy Standard”
    - Ver [Classic SysAdmin: The Linux Filesystem Hierarchy Standard Explained](https://www.linuxfoundation.org/blog/blog/classic-sysadmin-the-linux-filesystem-explained)
- [How to install Linux Mint 17 on SSD and have Home on HDD](https://unix.stackexchange.com/questions/155435/how-to-install-linux-mint-17-on-ssd-and-have-home-on-hdd)

