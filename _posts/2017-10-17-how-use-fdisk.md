---
title: "Как добавить новый диск в систему"
layout: post
categories: Other
tags: linux centos fdisk
---

>OC: CentOS 6.8(Final)

Подключаем диск и набираем ` fdisk -l`.
```
# fdisk -l

Disk /dev/xvda: 8589 MB, 8589934592 bytes
255 heads, 63 sectors/track, 1044 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x000df76f

Device Boot Start End Blocks Id System
/dev/xvda1 * 1 64 512000 83 Linux
Partition 1 does not end on cylinder boundary.
/dev/xvda2 64 1045 7875584 8e Linux LVM

Disk /dev/mapper/VolGroup-lv_root: 7205 MB, 7205814272 bytes
255 heads, 63 sectors/track, 876 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000


Disk /dev/mapper/VolGroup-lv_swap: 855 MB, 855638016 bytes
255 heads, 63 sectors/track, 104 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000


Disk /dev/xvdb: 10.7 GB, 10737418240 bytes
255 heads, 63 sectors/track, 1305 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000
```

Новый диск появился как `/dev/xvdb`.
```
# fdisk /dev/xvdb

Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0xd85212d9.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.

Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
switch off the mode (command 'c') and change display units to
sectors (command 'u').

Command (m for help): m
Command action
a toggle a bootable flag
b edit bsd disklabel
c toggle the dos compatibility flag
d delete a partition
l list known partition types
m print this menu
n add a new partition
o create a new empty DOS partition table
p print the partition table
q quit without saving changes
s create a new empty Sun disklabel
t change a partition \'\s system id
u change display/entry units
v verify the partition table
w write table to disk and exit
x extra functionality (experts only)
```

Создаем раздел используя всё пространство на жестком диске: команда `n`, затем `p` и `1`, значения первого и последнего цилиндра оставить по умолчанию. После окончания разметки записываем изменения командой `w`
```
Command (m for help): n
Command action
e extended
p primary partition (1-4)
p
Partition number (1-4): 1
First cylinder (1-1044, default 1): 1
Last cylinder or +size or +sizeM or +sizeK (1-1044, default 1044):
Using default value 1044

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

Проверяем:
```
# fdisk -l /dev/xvdb

Disk /dev/xvdb: 10.7 GB, 10737418240 bytes
255 heads, 63 sectors/track, 1305 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xd85212d9

Device Boot Start End Blocks Id System
/dev/xvdb1 1 1305 10482381 83 Linux
```

Форматируем в ext3
```
# mkfs -t ext3 /dev/xvdb1

mke2fs 1.41.12 (17-May-2010)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
655360 inodes, 2620595 blocks
131029 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2684354560
80 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 28 mounts or
180 days, whichever comes first. Use tune2fs -c or -i to override.
```

Подключаем
```
# mkdir /mnt/xvdb
# mount -t ext3 /dev/xvdb1 /mnt/xvdb
```

Проверяем
```
# df -h
Filesystem Size Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root 6.5G 922M 5.3G 15% /
tmpfs 249M 0 249M 0% /dev/shm
/dev/xvda1 477M 71M 382M 16% /boot
/dev/xvdb1 9.9G 151M 9.2G 2% /mnt/xvdb
```

Добавляем в fstab (`joe /etc/fstab`)
```
/dev/xvdb1 /mnt/xvdb ext3 defaults 0 0
```

И после перезагрузки проверяем автоматическое монтирование командой `# df -h`.