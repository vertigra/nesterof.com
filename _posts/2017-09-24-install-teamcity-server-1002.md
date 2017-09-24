---
title: Как установить TeamCity Server 10.0.2
layout: post
categories: TeamCity
tags: teamcity java debian linux
---

>OC: Debian 8 Jessie

Хорошо написано [здесь](http://maxim.rubchinsky.com/install-teamcity-ubuntu/).

[Ставим Java](https://nesterof.com/blog/2017/09/23/install-java-debian/) и скачиваем TeamCity
```
# cd /opt
# wget https://download.jetbrains.com/teamcity/TeamCity-10.0.2.tar.gz
# tar -xvf TeamCity-10.0.2.tar.gz
# chown -R $user-name TeamCity
```

Добавляем скрипт автозапуска в `/etc/init.d/teamcity` (меняем $user-name на существующий в системе)
```bash
#!/bin/sh
### BEGIN INIT INFO
# Provides: teamcity
# Required-Start:
# Required-Stop:
# Should-Start:
# Should-Stop:
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: team-city startup script
### END INIT INFO

export TEAMCITY_DATA_PATH="/opt/TeamCity/bin/.BuildServer"

case $1 in
start)
start-stop-daemon --start -c $user-name --exec /opt/TeamCity/bin/runAll.sh start
;;

stop)
start-stop-daemon --start -c $user-name --exec /opt/TeamCity/bin/runAll.sh stop
;;

esac

exit 0
```

Меняем права и ставим в автозапуск
```
# chmod +x /etc/init.d/teamcity
# update-rc.d /etc/init.d/teamcity defaults
```

Запускаем демона `# /etc/init.d/teamcity start`

Удаляем архив `# rm /opt/TeamCity-10.0.2.tar.gz`

Переходим на внешний адрес сервера `http://ip.server:8111` и жмем Proceed.

На странице Database connection setup выбираем тип базы данных - Internal(HSQLDB)

Заключительный этап - указать логин и пароль админимстратора.