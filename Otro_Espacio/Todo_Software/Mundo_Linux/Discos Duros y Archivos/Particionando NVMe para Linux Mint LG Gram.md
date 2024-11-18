# Cómo particioné el NVMe para instalar Linux Mint en el LG Gram 16

La conclusión la saqué luego de ver lo que sugieren en estos posts:

- [How to Install Linux Mint 21 with manual partitions](https://www.addictivetips.com/ubuntu-linux-tips/how-to-install-linux-mint-21-with-manual-partitions/)
- [Foro Linux Mint - Julio, 2021](https://forums.linuxmint.com/viewtopic.php?p=2043622&sid=eb0e8b623df96b8a6e4688b3cb59965e#p2043622)
- [Subreddit Linux Mint](https://www.reddit.com/r/linuxmint/comments/17rgnww/comment/k8jmva9/)

# Conclusión

Así quedó la tabla de particiones:

![[10.partitions.nvme.png]]

Versión solo texto:
```bash
/dev/nvme0n1p1 efi 300mb
/dev/nvme0n1p2 ext4 / 90gb
/dev/nvme0n1p3 swap 8gb
/dev/nvme0n1p4 ext4 /home 400gb
```