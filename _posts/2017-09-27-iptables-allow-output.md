---
title: "Как разрешить в iptables исходящий трафик для определенного порта"
layout: post
categories: Iptables
tags: iptables debian linux
---

>OC: Debian 8 Jessie

Если конфигурацией фаервола запрещен исходящий трафик

```
-A INPUT -j REJECT
-A FORWARD -j REJECT
-A OUTPUT -j REJECT
```

как [здесь](https://nesterof.com/blog/2017/09/27/iptables-setting-openvpn/), то разрешить **исходящий** трафик можно так:

```
# Allow TCP output traffic on port 2500.
-A INPUT -i eth0 -p tcp -m state --state ESTABLISHED --sport 2500 -j ACCEPT
-A OUTPUT -o eth0 -p tcp -m state --state NEW,ESTABLISHED --dport 2500 -j ACCEPT
```

Эта цепочка правил разрешит исходящие соединения (и ответный входящий трафик) на порту 2500. Например исходящие SSH соединения на порт 2500.

При этом, если на порту 2500 висит сетевая служба (`LISTEN`) то она по-прежнему останется недоступной(закрытой) для указного сетевого интерфейса.

А такая цепочка наоборот откроет входящие соединения на порт 10206.

```
# Allow UDP input traffic on port 10206.
-A INPUT -i venet0 -p udp -m state --state NEW,ESTABLISHED --dport 10206 -j ACCEPT
-A OUTPUT -o venet0 -p udp -m state --state ESTABLISHED --sport 10206 -j ACCEPT
```