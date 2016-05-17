# Instalando proxmox 4.x

## Las características del servidor son:

Dell PowerEdge T605 Server
* 2x AMD Quad Core Opteron 2376 (2,3GHz, 6MB, 75W ACP)
* 32 GB RAM ECC REGISTERED
* 1x 256GB Western Digital SATA   
* 2x 2TB Western Digital RED SATA
* 1x 120GB SSD SATA Kingston 111 GB
* 1x SAS 6i/R controlador interno RAID PCI-E
* 4x NIC Gigabit Ethernet

## Instalación de Proxmox
Nota: Al instalar Proxmox, no se puede dejar la cantidad de espacio libre que queramos. Consecuentemente, en esta guía instalaremos el sistema base con unos LVM más pequeños de los necesarios y a continuación usaremos un LiveCD para encoger el tamaño de la partición del grupo de LVM

111GB Kingston SATA SSD - Sistema Base instalado en un partición ext4 + 8GB partición de log ZFS (ZIL) + 32GB partición caché ZFS (L2ARC)

        8GB  partición de Log.
        32GB partición caché ZFS.
        32GB / partición root
        16GB Partición Linux swap (ver nota debajo)
        32GB partición pve-data 

Este enfoque parece funcionar bastante bien, ¡pero asegurate de inicializar el valor de *vm.swappiness* a un valor bajo si tienes la partición de swap en un SSD! Incrementará el uso de la RAM un poco, pero es más facil tenerlo en el SSD y hace que la máquina vaya un poco más rápida. Normalmente el valor es 60, lo cual indica que cuando la RAM se llena al 60%, se empieza a paginar con el SSD. Podemos averiguar el valor actual con la orden:

```bash
    cat /proc/sys/vm/swappiness
```

Para cambiar el valor, debemos editar el fichero */etc/sysctl.conf* y añadir la línea:

```bash
    vm.swappiness = 10
```

Hay cinco opciones para inicializar el almacenamiento durante la instalación de Proxmox: 

    swapsize : tamaño de la swap de Linux.
    maxroot : Este es el tamaño de la partición / (root).
    minfree : Debería ser tu tamaño de log ZFS + cache ZFS. En mi disco SSD de 111GB, éste era 32+8=40.
    maxvz : Ésta es la partición pve-data al la que me refería arriba. 
    Filesystem : Déjalo en ext4 a menos que tengas una buena razón para no hacerlo.

![](imagenes/instalacion1.png)


Una vez hecho lo anterior, configura una contraseña y una zona horaria, y después configura la red. Debes asignar una IP fija que no debs cambiar por un largo periodo de tiempo, es bastante enredoso cambiar la IP. El FQDN no tiene que ser un nombre real (existente) en internet. Si no sabes cual poner, *servidor1.localdomain* será suficiente.

![](imagenes/instalacion2.png)

Una vez finalizada la instalación, ya podemos reiniciar, pero antes de esto, necesitamos encoger la partición de los LVM

![](imagenes/instalacion3.png)

## Reparando el particionamiento del SSD 

Usando un LiveCD, inicia gparted y cambia el tamaño de la partición del LVM

En las siguientes imágenes se muestra la secuencia de pasos:

![](imagenes/gparted1.png)

![](imagenes/gparted2.png)

![](imagenes/gparted3.png)

![](imagenes/gparted4.png)

## Configura la nueva instalación de Proxmox

Go ahead and boot back into Proxmox, but don't plug in your mechanical drives yet, only have the SSD hooked up.

Note: You can do everything from this point forward via SSH if you'd rather.

Once the machine boots, log in with root and the password you set during the install.
Fixing Proxmox repositories

Proxmox is intended for use in production environments with the purchase of a subscription from the company. If you have purchased a subscription, you should enter your key through the web interface and skip this step.

Note: Information on this step has been pulled from [the Proxmox documentation.](https://pve.proxmox.com/wiki/Packagerepositories#Proxmox_VE_No-Subscription_Repository) Feel free to read more about the repository system there._

Open your sources.list file with the text editor of your choice. You can use vi or vim, but if you're new to Linux you should use nano /etc/apt/sources.list.

Once you open it, add deb http://download.proxmox.com/debian jessie pve-no-subscription at the bottom. It's probably a good idea to leave a comment to note why you did this, but you don't have to. Your file should look something like this in the end.

Newbies: If you're using nano, press CTRL+X, Y for yes, and ENTER to save and exit.

You also need to remove the subscription-only repository from your APT sources. It's stored in it's own file, so go ahead and delete it by running rm /etc/apt/sources.list.d/pve-enterprise.list.

Once that's done, run apt-get update, then apt-get upgrade -y, and finally update-grub just in case. This will download all the updates you need, so it may take a while depending on your internet speed.
Making ZFS partitions on your SSD

We're going to use the handy command-line utility cfdisk to partition out the free space we made into ZFS log/cache partitions. You should have already chosen the size of these partitions; for this guide I am opting for an 8GB log partition and a 32GB cache partition.

Still logged in as root, run cfdisk /dev/sda. If you find cfdisk is not installed, you can install it with apt-get install cfdisk.

You can see the free space we made earlier highlighted in purple. The text-based UI is pretty self-explanatory, so go ahead and make the partitions you need. You can see below how I did mine from start to finish.


