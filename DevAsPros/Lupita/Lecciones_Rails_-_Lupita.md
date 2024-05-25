# Lecciones Rails - Lupita

## Baja velocidad de descarga de las *boxes* de Vagrant

Resulta que al intentar bajar la *box* `ubuntu/bionic64` desde el servidor oficial de Vagrant mediante los comandos `vagrant up` o `vagrant add [BOX]` la velocidad de descarga es supremamente lenta.

Hay un [issue](https://github.com/hashicorp/vagrant/issues/5319) donde se menciona que esto es ajeno a *Vagrant* pero tampoco hay una solución ni alternativa clara excepto por descargar “manualmente” usando comandos como `wget` o desde el navegador.

Luego de descargar por otro medio, [mover la](https://github.com/hashicorp/vagrant/issues/5319#issuecomment-345567579) [*box*](https://github.com/hashicorp/vagrant/issues/5319#issuecomment-345567579) [descargada a la ruta](https://github.com/hashicorp/vagrant/issues/5319#issuecomment-345567579) que Vagrant espera:


    $ wget https://vagrantcloud.com/ubuntu/boxes/bionic64/versions/20190720.0.0/providers/virtualbox.box
    
    $ vagrant box add ubuntu/bionic64 ../bionic64.box

También sugieren usar *boxes* que estén ubicados en otros servidores.

Finalmente, hay quienes sugieren Docker 🙂 

