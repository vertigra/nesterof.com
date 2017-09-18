---
title: "Как добавить Local Storage"
layout: post
categories: XenServer
tags: xenserver
---

> OC: XEN 6.5

Взято [отсюда](https://serveradmin.ru/kak-podklyuchit-zhestkiy-disk-v-xenserver/)

1. Подключаем диск и смотрим как он определился в системе:

   ```
   # fdisk -l 
   WARNING: GPT (GUID Partition Table) detected on /dev/sda! The util fdisk doesn't support GPT. Use GNU Parted.
       
   Disk /dev/sda: 250.0 GB, 250059350016 bytes
   256 heads, 63 sectors/track, 30282 cylinders
   Units = cylinders of 16128 * 512 = 8257536 bytes
   Device Boot Start End Blocks Id System
   /dev/sda1 * 1 30283 244198583+ ee EFI GPT
   
   WARNING: GPT (GUID Partition Table) detected on /dev/sdb! The util fdisk doesn't support GPT. Use GNU Parted.
   
   Disk /dev/sdb: 250.0 GB, 250059350016 bytes
   255 heads, 63 sectors/track, 30401 cylinders
   Units = cylinders of 16065 * 512 = 8225280 bytes
   Disk /dev/sdb doesn't contain a valid partition table
   
   Disk /dev/sdc: 250.0 GB, 250000000000 bytes
   255 heads, 63 sectors/track, 30394 cylinders
   Units = cylinders of 16065 * 512 = 8225280 bytes
   Disk /dev/sdc doesn't contain a valid partition table
   ```
   Диск определился как `/dev/sdc`

2. Удаляем все разделы (если есть) командой `# fdisk /dev/sdc`. Поcле чего нажимаем `d` и выбираем номер раздела который нужно удалить (1-4). Для записи изменений на диск нужно нажать `w`.

3. После чего выполним инициализацию диска для работы с LVM `# pvcreate /dev/sdb`

4. Смотрим UUID хоста гипервизора
```
# xe host-list
uuid ( RO): 039081cb-ecfe-4163-bf6e-992b1ef1a142
name-label ( RW): xenserver
name-description ( RW): Default install of XenServer
```

5. Теперь можно создать локальное хранилище
```
# xe sr-create content-type=user host-uuid=039081cb-ecfe-4163-bf6e-992b1ef1a142 type=lvm device-config-device=/dev/sdc name-label="Temp Local Storage"
```