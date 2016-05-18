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

(8GB Log ZFS + 32 GB Cache ZFS) = 40 * 1024 = 40960, por tanto encogeremos la partición 40960MB.

![](imagenes/gparted2.png)

![](imagenes/gparted3.png)

![](imagenes/gparted4.png)

## Configura la nueva instalación de Proxmox

Ahora ya podemos iniciar normalmente proxmox.
Algo que hay que cambiar es añadir un parámetro al arranque de grub. Es el parámetro **rootdelay=10**. Para ello editaremos el archivo /etc/grub/default y añadiremos el parámetro al en la línea *GRUB_CMDLINE_LINUX_DEFAULT="rootdelay=10 quiet"* del fichero. Después habrá que ejecutar:
```bash
   sudo update-grub /dev/sda
```

Nota: Podemos realizar todo lo que viene a continuación desde una sesión SSH

## Cambiar los repositorios de Proxmox
Proxmox está destinado para su uso en entornos de producción con la compra de una suscripción de la compañía . Si hemos adquirido una suscripción , debemos introducir nuestra clave a través de la interfaz web y omitir este paso.

Abre el fichero /etc/apt/sources.list con tu editor de texto preferido y añade al final la línea *deb http://download.proxmox.com/debian jessie pve-no-subscription*.

También debes eliminar el repositorio para suscriptores de los orígenes de APT. Es un fichero que puedes borrar con la orden : *rm /etc/apt/sources.list.d/pve-enterprise.list*

Una vez realizados los cambios, ejecuta las siguientes órdenes (una detrás de otra):

```bash
   apt-get update
   apt-get upgrade -y
   update-grub
```

Esto descargará todas las actualizaciones que necesita , lo que puede tardar un tiempo dependiendo de la conexión aInternet.

## Realizar las particiones ZFS en el disco SSD

Vamos a utilizar la utilidad *cfdisk* de línea de comandos para dividir el espacio libre que hicimos en las particiones ZFS  de log y de cache. En nuestro caso estoy optando por una partición de Log de 8 GB y una partición de caché ZFS de 32 GB.

Ejecuta *cfdisk /dev/sda*. 

Se puede ver el espacio libre que hicimos antes resaltado en color morado . La interfaz de usuario basada en texto es bastante explicativa por sí misma , así que adelante y hacer las particiones que necesitamos. Se puede ver a continuación cómo lo hice mío de principio a fin.

![](imagenes/cfdisk.png)


