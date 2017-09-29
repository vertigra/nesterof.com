---
title: Как управлять iptables
layout: post
categories: Iptables
tags: iptables debian linux
---

> OC: Debian 8 Jessie

### Сбросить правила iptables (IPv4 или IPv6)

```
# iptables -F && iptables -X
# ip6tables -F && ip6tables -X
```

### Посмотреть текущий статус iptables (IPv4 или IPv6)

```
# iptables -L -nv
# ip6tables -L -nv
```

### Применить новые правила (IPv4 или IPv6)

```
# iptables-restore < /etc/iptables/rules.v4
# ip6tables-restore < /etc/iptables/rules.v6
```

### Выключить iptables (IPv4)

```
# iptables-save > $HOME/firewall.txt
# iptables -X
# iptables -t nat -F
# iptables -t nat -X
# iptables -t mangle -F
# iptables -t mangle -X
# iptables -P INPUT ACCEPT
# iptables -P FORWARD ACCEPT
# iptables -P OUTPUT ACCEPT
```

### Выключить ip6tables (IPv6)

```
# ip6tables-save > $HOME/firewall-6.txt
# ip6tables -X
# ip6tables -t mangle -F
# ip6tables -t mangle -X
# ip6tables -P INPUT ACCEPT
# ip6tables -P FORWARD ACCEPT
# ip6tables -P OUTPUT ACCEPT
```