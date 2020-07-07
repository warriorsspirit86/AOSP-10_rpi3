Read it first : https://github.com/android-rpi/local_manifests/tree/android10     
    
## Build Kernel    
```
$ sudo apt install gcc-arm-linux-gnueabihf libssl-dev    
$ cd kernel/rpi    
$ ARCH=arm scripts/kconfig/merge_config.sh arch/arm/configs/bcm2709_defconfig kernel/configs/android-base.config kernel/configs/android-recommended.config    
$ ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make zImage    
$ ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make dtbs    
```
    
## Install python mako module    
```
 $ sudo apt install python-mako    
```
    
## Build Android source    
```
 Continue build with http://source.android.com/source/building.html    
 $ source build/envsetup.sh    
 $ lunch rpi3-eng    
 $ make ramdisk systemimage vendorimage    
```

## Prepare sd card    

Partitions of the card should be set-up like followings.    
p1 256MB for BOOT : Do fdisk : W95 FAT32(LBA) & Bootable, mkfs.vfat    
p2 640MB for /system : Do fdisk, new primary partition    
p3 128MB for /vendor : Do fdisk, new primary partition    
p4 remainings for /data : Do fdisk, mkfs.ext4     
    
```
$ lsblk    
$ sudo fdisk /dev/mmcblk0    
    Command (m for help): m
    Command (m for help): d
    Command (m for help): p
    Command (m for help): n
    Partition type
       p   primary (0 primary, 0 extended, 4 free)
       e   extended (container for logical partitions)
    Select (default p): 

    Using default response p.
    Partition number (1-4, default 1): 
    First sector (2048-30769151, default 2048): 
    Last sector, +sectors or +size{K,M,G,T,P} (2048-30769151, default 30769151): +256M
    Command (m for help): t
    Selected partition 1
    Hex code (type L to list all codes):  L
    ...    
     c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi ea  Rufus alignment
    ...
    Hex code (type L to list all codes): c
    Changed type of partition 'Linux' to 'W95 FAT32 (LBA)'.
    Command (m for help): p
   
   Device         Boot Start    End Sectors  Size Id Type
/dev/mmcblk0p1    2048 526335  524288  256M  c W95 FAT32 (LBA)

Command (m for help): n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (2-4, default 2): 
First sector (526336-30769151, default 526336): 
Last sector, +sectors or +size{K,M,G,T,P} (526336-30769151, default 30769151): +640M

Created a new partition 2 of type 'Linux' and of size 640 MiB.

Command (m for help): n       
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (3,4, default 3): 
First sector (1837056-30769151, default 1837056):             
Last sector, +sectors or +size{K,M,G,T,P} (1837056-30769151, default 30769151): +128M

Created a new partition 3 of type 'Linux' and of size 128 MiB.

Command (m for help): n
Partition type
   p   primary (3 primary, 0 extended, 1 free)
   e   extended (container for logical partitions)
Select (default e): p

Selected partition 4
First sector (2099200-30769151, default 2099200): 
Last sector, +sectors or +size{K,M,G,T,P} (2099200-30769151, default 30769151): 

Created a new partition 4 of type 'Linux' and of size 13.7 GiB.

Command (m for help): p
Disk /dev/mmcblk0: 14.7 GiB, 15753805824 bytes, 30769152 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xce361f12

Device         Boot   Start      End  Sectors  Size Id Type
/dev/mmcblk0p1         2048   526335   524288  256M  c W95 FAT32 (LBA)
/dev/mmcblk0p2       526336  1837055  1310720  640M 83 Linux
/dev/mmcblk0p3      1837056  2099199   262144  128M 83 Linux
/dev/mmcblk0p4      2099200 30769151 28669952 13.7G 83 Linux

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

## Write system & vendor partition    
```
  $ cd out/target/product/rpi3    
  $ sudo dd if=system.img of=/dev/<p2> bs=1M    
  $ sudo dd if=vendor.img of=/dev/<p3> bs=1M    
```
  
## Copy kernel & ramdisk to BOOT partition    
```
  device/brcm/rpi3/boot/* to p1:/    
  kernel/rpi/arch/arm/boot/zImage to p1:/    
  kernel/rpi/arch/arm/boot/dts/bcm2710-rpi-3-b.dtb to p1:/    
  kernel/rpi/arch/arm/boot/dts/overlays/vc4-kms-v3d.dtbo to p1:/overlays/vc4-kms-v3d.dtbo    
  out/target/product/rpi3/ramdisk.img to p1:/    
```
    
## HDMI_MODE : If DVI monitor does not work, try followings for p1:/config.txt    
```
  hdmi_group=2    
  hdmi_mode=85    
```
