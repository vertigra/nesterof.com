---
title: Как назначить права на директорию и файл БД Firebird
layout: post
categories: Firebird
tags: linux firebird debian
---

> OC: Debian 8 Jessie (x64)

Все давно написано [тут](http://www.firebirdfaq.org/faq102/).

* Добавить себя в группу firebird

```
# addgroup username firebird
```

* Права на директорию БД 770, пользователь и группа firebird

```
# chown -R firebird:firebird /databasedir (R - recursive)
# chmod 770 /databasedir
```

* Права на файл базы данных 660, пользователь и группа firebird

```
# chown firebird:firebird datbasefilename.fdb
# chmod 660 datbasefilename.fdb
```

Пример использования утилиты gbak (бэкап базы данных)

```
$
gbak -b -g -v -user SYSDBA -password masterkey datbasefilename.fdb testbackup.fbk
```