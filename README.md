# **Файловые системы и LVM**

## **Homework**

 **Работа с LVM**.
 
На имеющемся образе centos/7 - v. 1804.2
- Уменьшить том под / до 8G
- Выделить том под /home
- Выделить том под /var -  сделать в mirror
- /home - сделать том для снапшотов
- Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами ( на выбор)

Работа со снапшотами:
- сгенерить файлы в /home/
- снять снапшот
- удалить часть файлов
- восстановится со снапшота
- залоггировать работу можно с помощью утилиты script

## **1.Начало работы с LVM**
 
Используем команду lsblk и утилиту lvmdiskscan, чтобы определить какие устройства мы хотим использовать в качестве Physical Volumes для будущих Volume Groups:

[vagrant@otuslvm ~]$ lsblk

NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT

sda      8:0    0  40G  0 disk 
└─sda1   8:1    0  40G  0 part /

sdb      8:16   0  10G  0 disk 

sdc      8:32   0   2G  0 disk 

sdd      8:48   0   1G  0 disk 

sde      8:64   0   1G  0 disk 

[vagrant@otuslvm ~]$ sudo lvmdiskscan 

  /dev/sda1 [     <40.00 GiB] 

  /dev/sdb  [      10.00 GiB] 

  /dev/sdc  [       2.00 GiB] 

  /dev/sdd  [       1.00 GiB] 

  /dev/sde  [       1.00 GiB] 

  4 disks

  1 partition

  0 LVM physical volume whole disks

  0 LVM physical volumes

**Создадим PV, чтобы разметить диск для дальнейшего использования LVM:**

[vagrant@otuslvm ~]$ sudo pvcreate /dev/sdb

  Physical volume "/dev/sdb" successfully created.

**Далее создаем VG otus - первый уровень абстракции:**

[vagrant@otuslvm ~]$ sudo vgcreate otus /dev/sdb

  Volume group "otus" successfully created

[vagrant@otuslvm ~]$ sudo lvs

  LV   VG   Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert

  test otus -wi-a----- <8.00g     

**Создадим LV с именем test в размере 80 от свободного общего количества:**

[vagrant@otuslvm ~]$ sudo lvcreate -l+80%FREE -n test otus

  Logical volume "test" created.

**Информация о созданном LVM:**

[vagrant@otuslvm ~]$ sudo vgdisplay otus

  --- Volume group ---

  VG Name               otus

  System ID             

  Format                lvm2

  Metadata Areas        1

  Metadata Sequence No  2

  VG Access             read/write

  VG Status             resizable

  MAX LV                0

  Cur LV                1

  Open LV               0

  Max PV                0

  Cur PV                1

  Act PV                1

  VG Size               <10.00 GiB

  PE Size               4.00 MiB

  Total PE              2559

  Alloc PE / Size       2047 / <8.00 GiB

  Free  PE / Size       512 / 2.00 GiB

  VG UUID               VbJu5Z-h44S-Y9TV-g06v-1B3c-Ikes-qd9FMu


[vagrant@otuslvm ~]$ lsblk

NAME        MAJ:MIN RM SIZE RO TYPE MOUNTPOINT

sda           8:0    0  40G  0 disk 
└─sda1        8:1    0  40G  0 part /

sdb           8:16   0  10G  0 disk 
└─otus-test 253:0    0   8G  0 lvm  

sdc           8:32   0   2G  0 disk 

sdd           8:48   0   1G  0 disk 

sde           8:64   0   1G  0 disk 

**Создадим еще один LV из свободного места, но не экстентами, а абсолютным значением в мегабайтах:**

[vagrant@otuslvm ~]$ sudo lvcreate -L100M -n small otus

  Logical volume "small" created.

[vagrant@otuslvm ~]$ sudo lvs

  LV    VG   Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert

  small otus -wi-a----- 100.00m                                                    

  test  otus -wi-a-----  <8.00g 

**Создадим на LV файловую систему и смонтируем его**

[vagrant@otuslvm ~]$ sudo mkfs.ext4 /dev/otus/test

...

Allocating group tables: done                            

Writing inode tables: done                            

Creating journal (32768 blocks): done

Writing superblocks and filesystem accounting information: done 

[vagrant@otuslvm ~]$ sudo mkdir /data

[vagrant@otuslvm ~]$ sudo mount /dev/otus/test /data/

[vagrant@otuslvm ~]$ sudo mount | grep /data

/dev/mapper/otus-test on /data type ext4 (rw,relatime,seclabel,data=ordered)


