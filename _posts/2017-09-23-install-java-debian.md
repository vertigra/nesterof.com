---
title: Как установить Java
layout: post
categories: Java
tags: java debian linux
---

> OC: Debian 8 Jessie

Взято [отсюда](http://tecadmin.net/install-java-8-on-debian/)

### Установка Java

Добавить в /etc/apt/sources.list.d/java-8-debian.list
```bash
deb http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main
deb-src http://ppa.launchpad.net/webupd8team/java/ubuntu trusty main
```

А потом так (надо будет согласится с лиценизией)
```
# apt-key adv --keyserver keyserver.ubuntu.com --recv-keys EEA14886
# apt-get update
# apt-get install oracle-java8-installer
```

Проверить что установилось можно так
```
# java -version
java version "1.8.0_101"
Java(TM) SE Runtime Environment (build 1.8.0_101-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.101-b13, mixed mode)
```

Указываем использовать установленную версию по умолчанию `# apt-get install oracle-java8-set-default`

### Установка переменной JAVA_HOME

Смотрим путь до Java командой `update-alternatives --config java`  и копируем его в `/etc/environment`.

```
# echo "JAVA_HOME="/usr/lib/jvm/java-8-oracle" >> /etc/environment
```

Применяем изменения `$ source /etc/environment`

Проверяем изменения `$ echo $JAVA_HOME`