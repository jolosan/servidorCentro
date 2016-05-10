# Instalando proxmox 4.x

## Las características del servidor son:

Dell PowerEdge T605 Server
* 2x AMD Quad Core Opteron 2376 (2,3GHz, 6MB, 75W ACP)
* 32 GB RAM ECC REGISTERED
* 1x 256GB Western Digital SATA   
* 2x 2TB Western Digital RED SATA
* 1x 120GB SSD PCIe Plextor 120 GB
* 1x SAS 6i/R controlador interno RAID PCI-E
* 4x NIC Gigabit Ethernet

## Instalación de Proxmox
Note: When I installed Proxmox, I could not get the "leave x amount of free space" option to work. I tried...with 6 reinstall attempts. Consequently, in this guide, we'll be installing the base system with smaller-than-needed LVM volumes, then using an Ubuntu live CD with gparted to shrink the LVM group's partition down to size. It's weird, but it got the job done.

120GB Plextor SSD - Base OS install on EXT4 partition + 8GB ZFS log partition (ZIL) + 32GB ZFS cache partition (L2ARC)



        8GB ZFS Log partition : 8GB should be fine.
        32GB ZFS cache partition : if you have a 256GB SSD, try 64GB of cache.
        32GB / root partition
        16GB Linux swap partition (see disclaimer below)
        32GB pve-data partition
This layout seems to work pretty good for my needs, but be sure to set vm.swappiness to a low value if you have your swap file on an SSD! It'll increase RAM usage a bit, but it's easier on your SSD and makes your machine a little jumpier. This step is included in this guide later on.

There are 5 key options in the Proxmox storage setup:

    swapsize : Linux swap file size.
    maxroot : This is the size of the / (root) partition
    minfree : This should be your ZFS log + your ZFS cache size. In my 120GB SSD, this was 32+8=40.
    maxvz : This is the pve-data partition I refer to above. I wouldn't make this too big unless you know what you're doing.
    Filesystem : Leave this on ext4 unless you have a good reason not to.

Once you get done with this, configure a password and your timezone, and then you get to network settings. Set this to something that'll work long term; it's a total pain to change the IP. The FDQN does not have to be an actual domain name on the internet, so if you don't know, just say whateveryouwant.localdomain and you'll be okay.

Once this is done, it'll do some stuff and then drop you at this screen. Once you get here, you're done with the install! Go ahead and click reboot, but don't let it actually boot into the new Proxmox install yet.

Fixing the SSD partitioning

We're going to use an Ubuntu Live CD with GParted to create free space on the SSD. I used Ubuntu GNOME for this guide, but you can use any Linux live CD with a partitioning tool. Just download the ISO and boot from it.

Configure your new Proxmox install

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


