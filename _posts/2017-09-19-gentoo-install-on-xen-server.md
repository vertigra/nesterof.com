---
title: "Как установить Gentoo на Xen Server"
layout: post
categories: Gentoo
tags: xenserver gentoo
---

> OC: XEN 6.5

Установка Gentoo - процесс ебливый и муторный. Убедитесь что вы к нему готовы морально и физически. Как говорил Стани́слав Е́жи Лец: "Лучше шесть раза поставить Debian, чем один раз Gentoo"

---

***NOTE***    
*Для сборки ядра и модулей необходим раздел root не меньше 11 Гб*


***NOTE***    
*Конфигурация виртуальной машины на которую производилась инсталяция: процессор (Vendor: AuthenticAMD, Model: AMD Phenom(tm) II X4 945 Processor, Speed: 3013 MHz) - Number of vCPUs - 4 Topology - 4 sockets with 1 core per socket, vCPU priority - Normal; память 1 Gb; жесткий диск -12 Гб, position 0; сеть DHCP. Время установки 6-7 часов.*


***NOTE***    
*Устанавливалось с носителя: [gentoo-install-x86-minimal-20161122.iso](http://distfiles.gentoo.org/releases/x86/autobuilds/20161122/install-x86-minimal-20161122.iso). Версия установки - чистая консоль без поддержки X-сервера*


***NOTE***    
*Если установка прервалась ~~запоем~~ после смены корня системы, вернуться к нужному этапу можно загрузившись с live-cd и набрать:*
```
livecd ~ # swapon /dev/sda2
livecd ~ # mount /dev/sda3 /mnt/gentoo
livecd ~ # mount /dev/sda1 /mnt/gentoo/boot
livecd ~ # mount -t proc none /mnt/gentoo/proc
livecd ~ # mount --rbind /sys /mnt/gentoo/sys
livecd ~ # mount --rbind /dev /mnt/gentoo/dev
livecd ~ # chroot /mnt/gentoo /bin/bash
livecd / # source /etc/profile
livecd / # export PS1="(chroot) $PS1"
(chroot) livecd / #
```

---

По мотивам [этой](http://www.oldnix.org/install-gentoo-linux/) и [этой](http://www.r-notes.ru/administrirovanie/gentoo-linux/116-gentoo-tipovaya-ustanovka.html) статьи с [продолжением](http://www.r-notes.ru/administrirovanie/gentoo-linux/117-gentoo-tipovaya-ustanovka-chast-2.html).

### Подготовка (терминал live-cd)

Скачиваем minimal installation cd c сайта [gentoo.org](https://gentoo.org/downloads/) и пишем его на флэшку или диск (на xen`e просто монтируем ISO на дисковод виртуальной машины)

После загрузки с диска проверяем сеть (если адрес выдается по DHCP никаких действий более не требуется)    
```
livecd ~ # ping -c 3 google.com 
PING google.com (37.29.18.38) 56(84) bytes of data.
64 bytes from 37.29.18.38: icmp_seq=1 ttl=57 time=55.6 ms
64 bytes from 37.29.18.38: icmp_seq=2 ttl=57 time=50.6 ms
64 bytes from 37.29.18.38: icmp_seq=3 ttl=57 time=48.7 ms

--- google.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 48.738/51.676/55.615/2.907 ms
```

Меняем пароль суперпользователя 
```
livecd ~ # passwd
```

Запускаем демон sshd 
```
livecd ~ # rc-service sshd start
```

Смотрим внешний ip-адрес виртуальной машины и подключаемся к ней по ssh
```
livecd ~ # ifconfig
enp0s4: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
inet 10.254.253.104 netmask 255.255.255.0 broadcast 10.254.253.255
inet6 fe80::e4e1:62ff:fea6:2f9c prefixlen 64 scopeid 0x20<link>
ether e6:e1:62:a6:2f:9c txqueuelen 1000 (Ethernet)
RX packets 205 bytes 15596 (15.2 KiB)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 60 bytes 8013 (7.8 KiB)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 173

lo: flags=73<UP,LOOPBACK,RUNNING> mtu 65536
inet 127.0.0.1 netmask 255.0.0.0
inet6 ::1 prefixlen 128 scopeid 0x10<host>
loop txqueuelen 1 (Local Loopback)
RX packets 6 bytes 108 (108.0 B)
RX errors 0 dropped 0 overruns 0 frame 0
TX packets 6 bytes 108 (108.0 B)
TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0
```

### Разметка диска (ssh)

Так как машина тестовая, будет использоваться упрощенная разметка диска:
```
/boot - 100 Mb
/swap - 1 Gb
/root - 10.9 Gb
```
Хорошо написано про разметку с помощью fdisk [тут](http://www.oldnix.org/fdisk-linux/).    

Проверяем существующую разметку командой `p`
```
livecd ~ # fdisk /dev/sda

Welcome to fdisk (util-linux 2.26.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x9ebcbacc.

Command (m for help): p
Disk /dev/sda: 12 GiB, 12884901888 bytes, 25165824 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x9ebcbacc
```

Размечаем раздел /boot (100 мегабайт) командой `n`
```
Command (m for help): n
Partition type
p primary (0 primary, 0 extended, 4 free)
e extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (1-4, default 1):
First sector (2048-25165823, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-25165823, default 25165823): +100M

Created a new partition 1 of type 'Linux' and of size 100 MiB.
```
На все запросы fdisk оставляем значения по умолчанию, кроме запроса last sector, там пишем +100M.

Размечаем раздел под swap (1 гигабайт)
```
Command (m for help): n
Partition type
p primary (1 primary, 0 extended, 3 free)
e extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (2-4, default 2):
First sector (206848-25165823, default 206848):
Last sector, +sectors or +size{K,M,G,T,P} (206848-25165823, default 25165823): +1G

Created a new partition 2 of type 'Linux' and of size 1 GiB.
```
Так же оставляем все значения по умолчанию, в запросе last sector пишем +1G.

Размечаем раздел под root все оставшееся место (11.9 гигабайт)
```
Command (m for help): n
Partition type
p primary (2 primary, 0 extended, 2 free)
e extended (container for logical partitions)
Select (default p):

Using default response p.
Partition number (3,4, default 3):
First sector (2304000-25165823, default 2304000):
Last sector, +sectors or +size{K,M,G,T,P} (2304000-25165823, default 25165823):

Created a new partition 3 of type 'Linux' and of size 10.9 GiB.
```
Здесь на все запросы включая last sector оставляем значения по умолчанию.

Меняем тип раздела для swap - команда `t`
```
Command (m for help): t
Partition number (1-3, default 3): 2
Partition type (type L to list all types): 82

Changed type of partition 'Linux' to 'Linux swap / Solaris'.
```
Для раздела 2 (Partition number) указываем тип раздела (Partition type) 82.

Проверяем что получилось - команда `p`
```
Command (m for help): p
Disk /dev/sda: 12 GiB, 12884901888 bytes, 25165824 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x9ebcbacc

Device Boot Start End Sectors Size Id Type
/dev/sda1 2048 206847 204800 100M 83 Linux
/dev/sda2 206848 2303999 2097152 1G 82 Linux swap / Solaris
/dev/sda3 2304000 25165823 22861824 10.9G 83 Linux
```

Если все правильно, сохраняем - команда `w`
```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
```

### Форматирование диска (ssh)

Форматируем файловые системы. Для раздела /boot - ext2. Для раздела /root - ext4. Для раздела подкачки - swap.
```
livecd ~ # mkfs.ext2 -L boot /dev/sda1
livecd ~ # mkfs.ext4 -L root /dev/sda3
livecd ~ # mkswap -L swap /dev/sda2
```

Подключаем своп
```
livecd ~ # swapon /dev/sda2
```

### Монтирование разделов (ssh)

Монтируем созданые разделы. Точка монтирования `/mnt/gentoo`
```
livecd ~ # mount /dev/sda3 /mnt/gentoo
livecd ~ # mkdir /mnt/gentoo/boot
livecd ~ # mount /dev/sda1 /mnt/gentoo/boot
```

### Получение установочных файлов (ssh)

Скачиваем установочные файлы. Последняя версия на момент установки: stage3-amd64-20161117.tar.bz2
```
livecd ~ # cd /mnt/gentoo
livecd ~ # wget -c http://gentoo.ussg.indiana.edu/releases/x86/autobuilds/current-stage3-i686/stage3-i686-20161122.tar.bz2
```

Распаковываем
```
livecd gentoo # tar xvjpf stage3-*.tar.bz2
```

Выбираем ближайшее зеркало для установки
```
livecd gentoo # mirrorselect -i -o >>/mnt/gentoo/etc/portage/make.conf
```

Копируем файл с адересами днс-серверов
```
livecd gentoo # cp -L /etc/resolv.conf /mnt/gentoo/etc/
```

### Монтирование слежебных файловых систем (ssh)

Для доступа к устройствам компьютера после смены корня системы необходимо смонтировать файловые системы /dev, /sys и /proc в корень устанавливаемой системы
```
livecd gentoo # mount -t proc none /mnt/gentoo/proc
livecd gentoo # mount --rbind /sys /mnt/gentoo/sys
livecd gentoo # mount --rbind /dev /mnt/gentoo/dev
```

### Смена корня файловой системы (ssh)

Меням корень фаловой системы. Точка монтирования `/mnt/gentoo`
```
livecd gentoo # chroot /mnt/gentoo /bin/bash
```

Перегружаем настройки профиля
```
livecd gentoo # source /etc/profile
```

Меняем приглашение командной строки
```
livecd gentoo # export PS1="(chroot) $PS1"
(chroot) livecd / #
```

### Установка снимков дерева Portage (ssh)

Устанавливаем снимок дерева Portage
```
(chroot) livecd / # mkdir /usr/portage
(chroot) livecd / # mkdir -p /usr/portage/profiles
(chroot) livecd / # echo «gentoo» > /usr/portage/profiles/repo_name
(chroot) livecd / # emerge-webrsync
```

Обновляем снимок дерев
```
(chroot) livecd / # emerge --sync
```

### Выбор профиля установки (ssh)

Выбираем профиль для установки `[1]` из предложенных
```
(chroot) livecd / # eselect profile list
Available profile symlink targets:
[1] default/linux/x86/13.0 *
[2] default/linux/x86/13.0/selinux
[3] default/linux/x86/13.0/desktop
[4] default/linux/x86/13.0/desktop/gnome
[5] default/linux/x86/13.0/desktop/gnome/systemd
[6] default/linux/x86/13.0/desktop/kde
[7] default/linux/x86/13.0/desktop/kde/systemd
[8] default/linux/x86/13.0/desktop/plasma
[9] default/linux/x86/13.0/desktop/plasma/systemd
[10] default/linux/x86/13.0/developer
[11] default/linux/x86/13.0/systemd
[12] hardened/linux/x86
[13] hardened/linux/x86/selinux
[14] hardened/linux/musl/x86
[15] default/linux/uclibc/x86
[16] hardened/linux/uclibc/x86

(chroot) livecd / # eselect profile set 1
```

### Редактируем make.conf (ssh)

Основные опции сборки системы храняться в файле `/etc/portage/make.conf`. 

Файл make.conf по умолчанию
```bash
# These settings were set by the catalyst build script that automatically
# built this stage.
# Please consult /usr/share/portage/config/make.conf.example for a more
# detailed example.
CFLAGS="-O2 -march=i486 -pipe"
CXXFLAGS="${CFLAGS}"
# WARNING: Changing your CHOST is not something that should be done lightly.
# Please consult http://www.gentoo.org/doc/en/change-chost.xml before changing.
CHOST="i486-pc-linux-gnu"
# These are the USE and USE_EXPAND flags that were used for
# buidling in addition to what is provided by the profile.
USE="bindist"
PORTDIR="/usr/portage"
DISTDIR="${PORTDIR}/distfiles"
PKGDIR="${PORTDIR}/packages"

GENTOO_MIRRORS="rsync://gentoo.bloodhost.ru/gentoo-distfiles ftp://gentoo.bloodhost.ru/ http://gentoo.bloodhost.ru/ ftp://xeon.gentoo.ru/mirrors/gentoo/distfiles/ ftp://mirror.yand
ex.ru/gentoo-distfiles/ http://mirror.yandex.ru/gentoo-distfiles/"
```

Значение переменных
```
CFLAGS и CXXFLAGS - параметры оптимизации компилятора gcc для языков C (CFLAGS) и C++ (CXXFLAGS)
CHOST - указывает компилятору gcc для какой архитектуры процессора собирать код
USE - указывает глобальные опции сборки ПО, полный список команда # less /usr/portage/profiles/use.desc
PORTDIR - расположение дерева Portage
DISTDIR - каталог для хранения сжатых исходных кодов
PKGDIR - каталог для хранения сжатых установочных бинарных пакетов
GENTOO_MIRRORS - зеркала, выбираются после команды
```

---

***NOTE***
[Здесь](http://gentoo-en.vfose.ru/wiki/User:Pepoluan/Paravirtualized_Gentoo_VMs_on_XenServer) рекомендуют следующее:
> For performance reasons, is strongly recommended to use the following as the value for CFLAGS:
>
> x86 DomU, Intel host: "-O2 -march=nocona -pipe -fomit-frame-pointer -mno-tls-direct-seg-refs"
> amd64 DomU, Intel host: "-O2 -march=nocona -pipe -fomit-frame-pointer"
> x86 DomU, AMD host: "-O2 -march=k8 -pipe -fomit-frame-pointer -mno-tls-direct-seg-refs"
> amd64 DomU, AMD host: "-O2 -march=k8 -pipe -fomit-frame-pointer"

Так как платформа у меня AMD выбираем `CFLAGS="-O2 -march=k8 -pipe -fomit-frame-pointer -mno-tls-direct-seg-refs"`

---

Редактируем файл make.conf `nano /etc/portage/make.conf` и дописываем следующие флаги `USE="-X -gtk -gtk2 -qt -qt4 -gnome -kde -xinetd unicode bindist"`. Это флаги отключат поддержку X сервера, xinetd, запретит собирать библиотеки для kde и gnome и добавит поддержку unicode.

Добавляем переменную `MAKEOPTS="-j5"` (на виртуальную машину выделено четыре ядра). Предназначена она для контроля запускаемых процессов компиляции при сборке пакета. Рекомендуется устанавливать ее значение исходя из количества ядер процессора плюс 1.

Файл `make.conf` после редактирования
```bash
# These settings were set by the catalyst build script that automatically
# built this stage.
# Please consult /usr/share/portage/config/make.conf.example for a more
# detailed example.
CFLAGS="-O2 -march=k8 -pipe -fomit-frame-pointer -mno-tls-direct-seg-refs"
CXXFLAGS="${CFLAGS}"
# WARNING: Changing your CHOST is not something that should be done lightly.
# Please consult http://www.gentoo.org/doc/en/change-chost.xml before changing.
CHOST="i686-pc-linux-gnu"
# These are the USE and USE_EXPAND flags that were used for
# buidling in addition to what is provided by the profile.
USE="-X -gtk -gtk2 -qt -qt4 -gnome -kde -xinetd unicode bindist"
PORTDIR="/usr/portage"
DISTDIR="${PORTDIR}/distfiles"
PKGDIR="${PORTDIR}/packages"
MAKEOPTS="-j5"

GENTOO_MIRRORS="rsync://gentoo.bloodhost.ru/gentoo-distfiles ftp://gentoo.bloodhost.ru/ http://gentoo.bloodhost.ru/ ftp://xeon.gentoo.ru/mirrors/gentoo/distfiles/ ftp://mirror.yand
ex.ru/gentoo-distfiles/ http://mirror.yandex.ru/gentoo-distfiles/
```

### Настройки времени и локали (ssh)

Выбираем часовой пояс `# ls /usr/share/zoneinfo`, находим свое местоположение и копируем
```
(chroot) livecd / # cp /usr/share/zoneinfo/Europe/Moscow /etc/localtime
(chroot) livecd / # echo "Europe/Moscow" > /etc/timezone
```

В файл `/etc/locale.gen` добавляем локаль
```
(chroot) livecd / # echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
```

Генерируем
```
(chroot) livecd / # locale-gen
* Generating locale-archive: forcing # of jobs to 1
* Generating 2 locales (this might take a while) with 1 jobs
* (1/2) Generating en_US.UTF-8 ... [ ok ]
* Generation complete
```

Смотрим LANG variables
```
(chroot) livecd / # eselect locale list
Available targets for the LANG variable:
[1] C
[2] POSIX
[3] en_US.utf8
[ ] (free form)
```

Выбираем `[3]`
```
(chroot) livecd / # eselect locale set 3
```

Добавим подгрузку шрифта при загрузке
```
(chroot) livecd / # rc-update add consolefont default
```

### Установка ядра (console)

Устанавливаем исходные коды ядра
```
(chroot) livecd / # emerge gentoo-sources
```

Собираем и устанавливаем ядро genkernel
```
(chroot) livecd / # emerge genkernel
```

Настраиваем /etc/genkernel.conf
```
(chroot) livecd / # nano /etc/genkernel.conf
```

Меняем значение следующих опций:
```
# Run 'make menuconfig' before compiling this kernel?
MENUCONFIG="yes"

# Make symlinks in BOOTDIR automatically?
SYMLINK="yes"

# Add new kernel to grub?
#BOOTLOADER="grub"
```
Запускаем сборку ядра:
```
(chroot) livecd / # genkernel --install all
```

---

***NOTE***     
**Минимальная необходимая конфигурация для XEN (без этого не запустится)**
```
Linux/x86 4.4.26-gentoo Kernel Configuration
--------------------------------------------
[]64-bit kernel
Device driver --->
Graphics support --->
Change <M> on <*> here
<*> Direct Renderinng Manager (XFree86 4.1.0 and higher DRI support --->
Set * here
<*> Cirrus driver for QUEMU emulated device
```

---

### Установка программ (console)

Устанавливаем необходимый набор программ
```
(chroot) livecd / # emerge udev syslog-ng dhcpcd vixie-cron grub
```
 * grub - загрузчик
 * udev - менеджер устройств для новых версий ядра Linux, подробнее [тут](https://ru.wikipedia.org/wiki/Udev) 
 * syslog-ng - открытая реализация протокола Syslog для Unix и Unix-подобных систем, подробнее [тут](https://redhat-club.org/2011/%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-syslog-ng-%D0%B4%D0%BB%D1%8F-%D1%86%D0%B5%D0%BD%D1%82%D1%80%D0%B0%D0%BB%D0%B8%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D0%BE%D0%B3%D0%BE-%D1%81%D0%B1%D0%BE%D1%80%D0%B0-%D0%B8-%D1%85%D1%80%D0%B0%D0%BD%D0%B5%D0%BD%D0%B8%D1%8F-%D1%81%D0%B8%D1%81%D1%82%D0%B5%D0%BC%D0%BD%D1%8B%D1%85-%D1%81%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D0%B9)
 * dhcpcd — свободная реализация клиента DHCP и DHCPv6. На данный момент является наиболее развитым DHCP-клиентом с открытым исходным кодом, подробнее [тут](http://roy.marples.name/projects/dhcpcd/index)
 * vixie-cron одна из реализаций программы cron, подробнее [тут](https://wiki.gentoo.org/wiki/Cron/ru#vixie_cron)

Добавляем в автозагрузку
```
(chroot) livecd / # rc-update add udev boot
(chroot) livecd / # rc-update add syslog-ng default
(chroot) livecd / # rc-update add vixie-cron default
```

### Редактируем fstab (console)

В файле `fstab` указываем точки монтрования файловых систем в соответствии с разметкой диска
```
# /etc/fstab: static file system information.
#
# noatime turns off atimes for increased performance (atimes normally aren't
# needed); notail increases performance of ReiserFS (at the expense of storage
# efficiency). It's safe to drop the noatime options if you want and to
# switch between notail / tail freely.
#
# The root filesystem should have a pass number of either 0 or 1.
# All other filesystems should have a pass number of 0 or greater than 1.
#
# See the manpage fstab(5) for more information.
#

# <fs> <mountpoint> <type> <opts> <dump/pass>

# NOTE: If your BOOT partition is ReiserFS, add the notail option to opts.
/dev/sda1 /boot ext2 noatime 1 2
/dev/sda2 / ext4 noatime 0 1
/dev/sda3 none swap sw 0 0
/dev/cdrom /mnt/cdrom auto noauto,ro 0 0
(required condition - empty string in end of file)
```

### Устанавливаем пароль root (console)

Устанавливаем пароль суперпользователя
```
(chroot) livecd init.d # passwd root
```

### Установка загрузчика (console)

Устанавливаем загрузчик
```
(chroot) livecd /# grep -v rootfs /proc/mounts > /etc/mtab
(chroot) livecd /# grub-install /dev/sda
Installing for i386-pc platform.
Installation finished. No error reported.
```

Генерируем файл grub.cfg
```
(chroot) livecd /# grub-mkconfig -o /boot/grub/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/kernel-genkernel-x86-4.4.26-gentoo
Found initrd image: /boot/initramfs-genkernel-x86-4.4.26-gentoo
done
```

### Финальная перезагрузка

Размонтируем файловые системы и перегрузимся
```
(chroot) livecd /# exit
livecd # umount -l /mnt/gentoo/dev{/shm,/pts,}
livecd # umount -l /mnt/gentoo/boot
livecd # umount -l /mnt/gentoo
livecd # reboot
```