---
title: Как установить Pritunl (Панель управления OpenVPN сервером)
layout: post
categories: OpenVPN
tags: linux, openvpn
---

> OC: Debian 8 Jessie*

Все собствено написано [тут](https://docs.pritunl.com/v1/docs/installation)

Добавить в `/etc/apt/sources.list.d/mongodb-org-3.2.list`

```
deb http://repo.mongodb.org/apt/debian wheezy/mongodb-org/3.2 main
```

А в `/etc/apt/sources.list.d/pritunl.list`

```
deb http://repo.pritunl.com/stable/apt jessie main
```

Затем смело ~~набирать~~ копировать и вставлять:

```
# apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 42F3E95A2C4F08279C4960ADD68FA50FEA312927
# apt-key adv --keyserver hkp://keyserver.ubuntu.com --recv 7568D9BB55FF9E5287D586017AE645C0CF8E292A
# apt-get update
# apt-get install pritunl mongodb-org
# systemctl start mongod pritunl
# systemctl enable mongod pritunl
```

И заходить браузером на 127.0.0.1 или внешний ip сервера (сервис висит на порту 443, дополнительно указывать это не нужно). 

Что бы избавиться от предупреждения "SEC_ERROR_UNKNOWN_ISSUER" добавляем для сайта иключение безопасности (как это сделать у мозиллы [тут](https://support.mozilla.org/ru/kb/kak-ustranit-oshibku-s-kodom-sec_error_unknown_iss))

Командой `pritunl setup-key` в командной строке сервера получаем ключ инсталяции и копируем его в поле "Enter setup key" веб интерфеса. После окончания установки можно заходить в веб-интерфейс. Пароль и логин по умолчанию `pritunl`.

Жрет вся эта красота гигабайт оперативки поэтому ~~на хуй не нужна~~ не совсем разумна ради пары клиентов на VDS.