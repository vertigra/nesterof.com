---
title: Как установить клиент OpenVPN на Debian 8 Jessie
layout: post
categories: OpenVPN
tags: linux, openvpn
---

> OC: Debian 8 Jessie

Устанавливаем OpenVPN и копируем заранее подготовленые сертификаты в /etc/openvpn

```
# apt-get install openvpn
```

Редактируем конфигурацию клиента

```
# joe client.conf
```

Меняем пути до сертификатов, указываем куда класть (~~a не ложить~~) логи. Указываем что OpenVPN следует запускать от пользователя nobody.

```
log-append /var/log/openvpn/client.log
status /var/log/openvpn/status.log

user nobody
group nogroup
```

Создаем дирректорию для логов

```
# mkdir /var/log/openvpn
# systemctl enable openvpn
```

Перезапускаем сервис

```
# service openvpn restart
```

(~~Ебемся, выясняя что ему еще нужно, потому как с первого раза никогда не запускается~~)