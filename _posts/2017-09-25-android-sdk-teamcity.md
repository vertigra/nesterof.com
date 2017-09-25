---
title: Как установить Android SDK на TeamCity агент
layout: post
categories: TeamCity
tags: teamcity java debian linux
---

>OC: Debian 8 Jessie

Скачать и распаковать свежий sdkmanager
```
# cd /opt
# wget https://dl.google.com /android/repository/tools_r25.2.3-linux.zip
# unzip tools_r25.2.3-linux.zip
# mv tools android-sdk-linux
```

Установить API, build-tools, platform-tools и прочее. 
```
# cd android-sdk-linux/tools
# ./android update sdk --no-ui
```

Если со свободным местом на жестком диске проблема можно ставить выборочно под проект. Для этого нужно сначала вывести весь список доступных пакетов командой `./android list sdk -u -a` и затем установить нужные `android update sdk --no-ui --filter 3,5,8,14` (например 3, 5, 8, 14)

Прописать переменную ANDROD_HOME в скрипт автозапуска TeamCity `/etc/init.d/teamcity`
```
export ANDROID_HOME="/opt/android-sdk-linux"
```

и в конце
```
# service teamcity restart
```