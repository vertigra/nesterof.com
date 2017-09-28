---
title: "Как настроить iptables после инсталяции"
layout: post
categories: Iptables
tags: iptables debian linux
---

> OC: Debian 8 Jessie

Для загрузки правил после перезапуска `#apt-get install iptables-persistent`

Копируем правила для IPv4 в `/etc/iptables/rules.v4`
```bash
*filter

# Allow all loopback (lo0) traffic and reject traffic
# to localhost that does not originate from lo0.
-A INPUT -i lo -j ACCEPT
-A INPUT ! -i lo -s 127.0.0.0/8 -j REJECT

# Allow ping.
-A INPUT -p icmp -m state --state NEW --icmp-type 8 -j ACCEPT

# Allow SSH connections.
-A INPUT -p tcp --dport 22 -m state --state NEW -j ACCEPT

# Allow inbound traffic from established connections.
# This includes ICMP error returns.
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Log what was incoming but denied (optional but useful).
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables_INPUT_denied: " --log-level 7

# Reject all other inbound.
-A INPUT -j REJECT

# Log any traffic that was sent to you
# for forwarding (optional but useful).
-A FORWARD -m limit --limit 5/min -j LOG --log-prefix "iptables_FORWARD_denied: " --log-level 7

# Reject all traffic forwarding.
-A FORWARD -j REJECT

COMMIT
```

**После `COMMIT` должна быть `newline` (переходим в конец файла и жмем `Enter`)**

Применяем правила `iptables-restore < /etc/iptables/rules.v4`

Копируем правила для IPv6  в `/etc/iptables/rules.v6`
```bash
*filter

# Allow all loopback (lo0) traffic and reject traffic
# to localhost that does not originate from lo0.
-A INPUT -i lo -j ACCEPT
-A INPUT ! -i lo -s ::1/128 -j REJECT

# Allow ICMP
-A INPUT -p icmpv6 -j ACCEPT

# Allow inbound traffic from established connections.
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Log what was incoming but denied (optional but useful).
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "ip6tables_INPUT_denied: " --log-level 7

# Reject all other inbound.
-A INPUT -j REJECT

# Log any traffic that was sent to you
# for forwarding (optional but useful).
-A FORWARD -m limit --limit 5/min -j LOG --log-prefix "ip6tables_FORWARD_denied: " --log-level 7

# Reject all traffic forwarding.
-A FORWARD -j REJECT

COMMIT
```

**Такая же история с `newline`**

Применяем правила `ip6tables-restore < /etc/iptables/rules.v6`