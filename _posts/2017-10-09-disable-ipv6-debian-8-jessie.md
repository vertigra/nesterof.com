---
title: Как отключить IPv6 в Debian 8 Jessie
layout: post
categories: Other
tags: debian linux ipv6
---

>OC: Debian 8 Jessie

Копируем в /etc/sysctl.d/99-sysctl.conf
```
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
net.ipv6.conf.eth0.disable_ipv6 = 1
net.ipv6.conf.<eth>.disable_ipv6 = 1 (указать все сетевые интерфейсы)
```

Применяем изменения `# sysctl -p`

Коментируем в /etc/hosts
```
#::1 localhost ip6-localhost ip6-loopback
```

Запрещаем весь траффик на IPv6 интерфейсе. В /etc/iptables/rules.v6 копируем
```
-A INPUT -j REJECT
-A FORWARD -j REJECT
-A OUTPUT -j REJECT
COMMIT
```

Применяем правила и проверяем.
```
# ip6tables-restore < /etc/iptables/rules.v6
# ip6tables -vL for v6
```

Так же надо знать что:

**APT attempts to resolve mirror domains to IPv6 as a result of apt-get update. If you choose to entirely disable and deny IPv6, this will slow down the update process for Debian and Ubuntu because APT waits for each resolution to time out before moving on.**

```
# joe /etc/gai.conf.
(uncomment)
precedence ::ffff:0:0/96 100
```

**UPD** [Здесь](http://vacadem.ru/blog/linux-unix-and-other/disable-ipv6-in-debian.html) пишут что надо еще так:

> В файле  /etc/netconfig закомментировать строки
```
#udp6 tpi_clts v inet6 udp - -
#tcp6 tpi_cots_ord v inet6 tcp - -
```

> В файле /etc/hosts закомментировать строки, относящиеся к IPv6
```
# The following lines are desirable for IPv6 capable hosts
#::1 ip6-localhost ip6-loopback
#fe00::0 ip6-localnet
#ff00::0 ip6-mcastprefix
#ff02::1 ip6-allnodes
#ff02::2 ip6-allrouters
```

> В /etc/ssh/ssh_config
```
AddressFamily inet
```

> В /etc/ssh/sshd_config раскомментировать строку ListenAddress для ipv4
```
#ListenAddress ::
ListenAddress 0.0.0.0
```

> В /etc/exim4/update-exim4.conf
```
disable_ipv6 = true
```

> Перегрузить и убедиться что у интерфейсов нет адреса IPv6

### Лирика

> Перегрузить и убедиться что у интерфейса нет адреса IPv6.

У интерфейсов нет адреса уже после внесения изменений в /etc/sysctl.d/99-sysctl.conf (первый этап), о чем кстати сказано на [официальном сайте](https://wiki.debian.org/DebianIPv6). А команда `# netstat --all | grep -E "tcp6|udp6"` выдает следующее (это дефолтная инсталляция Debian, с минимальными настройками):
```
tcp6 0 0 [::]:40916 [::]:* LISTEN
tcp6 0 0 [::]:ssh [::]:* LISTEN
tcp6 0 0 [::]:sunrpc [::]:* LISTEN
udp6 0 0 [::]:41176 [::]:*
udp6 0 0 [::]:1020 [::]:*
udp6 0 0 [::]:11286 [::]:*
udp6 0 0 [::]:sunrpc [::]:*
```

После выполнения первой части (до UDP):
```
# netstat --all | grep -E "tcp6|udp6"
tcp6 0 0 [::]:ssh [::]:* LISTEN
tcp6 0 0 [::]:50411 [::]:* LISTEN
tcp6 0 0 [::]:sunrpc [::]:* LISTEN
udp6 0 0 [::]:48355 [::]:*
udp6 0 0 [::]:6899 [::]:*
udp6 0 0 [::]:621 [::]:*
udp6 0 0 [::]:sunrpc [::]:*
```

Tо есть сервисы надо отключить.

После выполнения действий написанных после UPD останется dhclient на udp6:
```
# netstat -tulpn | grep -E "tcp6|udp6"
udp6 0 0 :::48384 :::* 426/dhclient
```

[Здесь](https://serveradmin.ru/nastroyka-seti-v-debian/) и [здесь](https://lists.debian.org/debian-user/2014/01/msg00234.html) написано как отключить:

>For dhclient edit /etc/dhcp/dhclient.conf and replace:- request subnet-mask, broadcast-address, time-offset, routers,
domain-name, domain-name-servers, domain-search, host-name, dhcp6.name-servers, dhcp6.domain-search, netbios-name-servers, netbios-scope, interface-mtu,
rfc3442-classless-static-routes, ntp-servers; 
with:- request subnet-mask, broadcast-address, time-offset, routers, domain-name, domain-name-servers, domain-search, host-name,
netbios-name-servers, netbios-scope, interface-mtu, rfc3442-classless-static-routes, ntp-servers;

В первом случае это не помогло (как и мне) dhclient остался висеть на ipv6 порту, это не страшно, запрашивать по ipv6 он ничего не будет.

Во втором вроде помогло (может из-за **Wheezy**):

> The above examples work with Wheezy.

Правдоподобное объяснение [тут](http://www.linuxquestions.org/questions/linux-newbie-8/dhcp-ipv6-4175466992/)

> The list shows dhclient as listening on IPv6 socket ::/UDP/62879. As long as the kernel has IPv6 support, it may not be possible to prevent applications (especially if they're running as root) from binding to an IPv6 socket. Since none of your interfaces have IPv6 addresses this shouldn't be a problem, but it does seem a little odd.

> Does the port number stay the same if you restart dhclient? This bug report seems to indicate that there is an NSUPDATE-related issure with dhclient, causing it to listen on random high ports.