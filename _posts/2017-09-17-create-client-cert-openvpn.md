---
title: Как сделать сертификат клиентa OpenVPN
layout: post
categories: OpenVPN
tags: linux, openvpn
---

>OC: Debian 8 Jessie

Генерируем сертификаты

```
# cd /etc/openvpn/easy-rsa (или туда где лежит easy-rsa)
# source ./vars && ./build-key clientname
```

Добавляем привязку клиента к ip (если надо)

```
# echo 'ifconfig-push 10.10.100.13 10.10.100.14' > /etc/openvpn/ccd/clientname
```

Архивируем

```
# tar -C /etc/openvpn/easy-rsa/keys -cvzf /etc/openvpn/client1.tar.gz {ca.crt,clientname.crt,clientname.key,ta.key}
```

Копируем

```
$ sftp user@ip
sftp> cd /etc/openvpn
sftp> get client1.tar.gz
sftp> bye
```