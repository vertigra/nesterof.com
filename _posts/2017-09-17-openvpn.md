---
title: Как управлять OpenVPN демоном
layout: post
categories: OpenVPN
tags: linux, openvpn
---

> OC: Debian 8 Jessie

Автоматический запуск при старте

```
# systemctl enable openvpn.service
```

Запуск/остановка/перезапуск демона

```
# systemctl start openvpn.service
# systemctl stop openvpn.service
# systemctl restart openvpn.service
```

Мониторинг демона OpenVPN

```
# systemctl status openvpn*.service

● openvpn.service - OpenVPN service
Loaded: loaded (/lib/systemd/system/openvpn.service; enabled)
Active: active (exited) since Tue 2015-10-13 20:55:13 UTC; 14min ago
Process: 5698 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
Main PID: 5698 (code=exited, status=0/SUCCESS)
CGroup: /system.slice/openvpn.service

Oct 13 20:55:13 localhost systemd[1]: Stopping OpenVPN service...
Oct 13 20:55:13 localhost systemd[1]: Starting OpenVPN service...
Oct 13 20:55:13 localhost systemd[1]: Started OpenVPN service.

● openvpn@server.service - OpenVPN connection to server
Loaded: loaded (/lib/systemd/system/openvpn@.service; disabled)
Active: active (running) since Tue 2015-10-13 20:55:13 UTC; 14min ago
Process: 5709 ExecStart=/usr/sbin/openvpn --daemon ovpn-%i --status /run/openvpn/%i.status 10 --cd /etc/openvpn --config /etc/openvpn/%i.conf (code=exited, status=0/SUCCESS)
Main PID: 5725 (openvpn)
CGroup: /system.slice/system-openvpn.slice/openvpn@server.service
└─5725 /usr/sbin/openvpn --daemon ovpn-server --status /run/openvpn/server.status 10 --cd /etc/openvpn --config ...
```