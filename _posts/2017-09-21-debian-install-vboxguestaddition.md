---
title: "Как установить VboxGuestAddition"
layout: post
categories: Other
tags: vboxguestaddition debian linux
---

> OC: Debian 8 Jessie

```
# apt-get install build-essential module-assistant
# m-a prepare
```

Монтируем образ дополнений из меню VirtualBox: Устройства -> Подключить образ диска дополнений гостевой OC...
```
# mount /dev/cdrom /mnt
# sh /mnt/VBoxLinuxAdditions.run
```

При появлении ошибки ```Error: unable to find the sources of your current Linux kernel...``` следует выполнить
```
# apt-cache search linux-headers-$(uname -r)
# apt-get install linux-headers-$(uname -r)
```