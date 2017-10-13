---
title: "Как управлять файлами через командную строку"
layout: post
categories: Other
tags: linux debian
---

> OC: Debian 8 Jessie

Утилиты linux для управления передачей файлов через командную строку - `sftp`, `scp` и `rsync`. Rsync не входит в состав minimal дистрибутива. Устанавливается так `# apt-get install rsync`.

### SCP

Отправить файл с локальной директории в удаленную
```bash
$ scp file.tar.gz user@ip:/home/user
```

Отправить файл с локальной директории в удаленную через нестандартный порт SSH(2500)
```bash
$ scp -P 2500 file.tar.gz user@ip:/home/user
```

### SFTP

Забрать файл с удаленной директории
``` bash
$ sftp login@ip

sftp> cd /path-to-file
sftp> get filename
```

Отправить файл с локальной директории в удаленную
``` bash
$ sftp login@ip

sftp> lcd /path-to-file
sftp> put filename
```

Если порт нестандартный добавляем `-P port`. Например ` $ sftp -P 2500 login@ip`

### RHOST

Отправить **все** файлы с локальной директории в удаленную

```bash
$ rsync -rPe "ssh -p 2500 -l login" * 10.10.100.5:/home/user/
```

`*` - маска файла
`-r` рекурсия
`-P` прогресс-бар
`-e` в кавычках после `е` указываем нестандартный порт и логин.

Rsync способен на большее. Узнать об этом так `$ rsync -h`. Просто пиздец  на что способен.