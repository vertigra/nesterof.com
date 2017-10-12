---
title: "Как установить OpenVPN сервер"
layout: post
categories: OpenVPN
tags: linux openvpn debian
---

>OC: Debian 8 Jessie


## Настройка сервера

Ставим пакеты - сам сервер и утилиту для управления сертификатами
```
# apt-get install easy-rsa && apt-get install openvpn
```

Копируем файлы утилиты управления сертификатами в `/etc/openvpn` что бы далеко за ними не ходить.
```
# mkdir /etc/openvpn/easy-rsa/
# cp -r /usr/share/easy-rsa/* /etc/openvpn/easy-rsa/
# cd /etc/openvpn/easy-rsa/
```

Правим переменные которые запрашиваются при создании ключей `# joe vars`.
```
export KEY_COUNTRY="US"
export KEY_PROVINCE="CA"
export KEY_CITY="SanFrancisco"
export KEY_ORG="Fort-Funston"
export KEY_EMAIL="me@myhost.mydomain"
```

Загружаем переменные и генерируем корневой сертификат
```
# source ./vars
# ./clean-all
# ./build-ca
```

Генерируем ключи для сервера и клиента. Вместо `client1` стоит подставить осмысленное имя, например `hostname` клиента Linux, или `net-bios name` клиента Windows.
```
# ./build-key-server server
# ./build-key client1
```

Генерируем файл Диффи-Хелмана и ключ HMAC для защиты трафика от расшифровки DoS-атак и флуда
```
# ./build-dh
# openvpn --genkey --secret keys/ta.key
```

Создаем конфигурационный файл сервера `# joe /etc/openvpn/server.conf` и копируем конфигурацию:
```
# OpenVPN server port
port 41268 
proto udp
dev tun

ca     /etc/openvpn/easy-rsa/keys/ca.crt
cert   /etc/openvpn/easy-rsa/keys/server.crt
key    /etc/openvpn/easy-rsa/keys/server.key
dh     /etc/openvpn/easy-rsa/keys/dh2048.pem

tls-server 
tls-auth /etc/openvpn/easy-rsa/keys/ta.key 0 

cipher DES-EDE3-CBC
auth SHA512

user nobody
group nogroup

persist-key
persist-tun


client-to-client
client-config-dir ccd

# OpenVPN subnet

server 10.10.10.0 255.255.255.0
ifconfig-pool-persist ipp.txt

keepalive 10 120

comp-lzo adaptive
persist-key
persist-tun

status /var/log/openvpn-status.log
log-append /var/log/openvpn.log

verb 3
mute 20
```
Предполагается что ключи лежат в `/etc/openvpn/easy-rsa/keys/`. Логи будут писаться в `/var/log`. Подсеть OpenVPN `10.10.100.0 255.255.255.0`, а сам сервер будет принимать запросы на порт 41268.

Разрешаем трафик на интерфейсе `TUN` и открываем порт `41268`
```
# joe /etc/iptables/rules.v4
```

```bash
# Allow UDP for OpenVPN
-A INPUT -i venet0 -p udp -m state --state NEW,ESTABLISHED --dport 41268 -j ACCEPT
-A OUTPUT -o venet0 -p udp -m state --state ESTABLISHED --sport 41268 -j ACCEPT

# Allow traffic on the TUN interface.
-A INPUT -i tun0 -j ACCEPT
-A OUTPUT -o tun0 -j ACCEPT
```
`venet0` - внешний интерфейс сервера. `tun0` - интерфейс openvpn.

Правила `iptables` сбрасываем и применяем новые
```
# iptables -F && iptables -X
# iptables-restore < /etc/iptables/rules.v4
```

Включаем автоматический запуск OpenVPN при загрузке и запускаем демон.
```
# systemctl enable openvpn.service
# systemctl start openvpn.service
```

## Настройка клиента

Складываем ключи в архив и копируем на клиента (sftp, scp, rsync).
```
tar -C /etc/openvpn/easy-rsa/keys -cvzf /etc/openvpn/client1.tar.gz {ca.crt,clientname.crt,clientname.key,ta.key}
```
Конфигурационные файлы практически идеинтичны. В обоих примерах `;remote my-server-2 1194` меняем на `remote ip port` сервера которому будем подключаться.

### windows

Расширение файла `.ovpn`. Параметр `user nobody` и `group nogroup` отлючен - windows не поддерживает `Downgrade privileges after initialization`. Логи хранятся в директории клиента.
```bash
##############################################
# Sample client-side OpenVPN 2.0 config file #
# for connecting to multi-client server.    #
# This configuration can be used by multiple #
# clients, however each client should have  #
# its own cert and key files.               #
# On Windows, you might want to rename this #
# file so it has a .ovpn extension          #
##############################################
# Specify that we are a client and that we
# will be pulling certain config file directives
# from the server.
client

# Use the same setting as you are using on
# the server.
# On most systems, the VPN will not function
# unless you partially or fully disable
# the firewall for the TUN/TAP interface.
;dev tap
dev tun

# Windows needs the TAP-Win32 adapter name
# from the Network Connections panel
# if you have more than one. On XP SP2,
# you may need to disable the firewall
# for the TAP adapter.
;dev-node MyTap

# Are we connecting to a TCP or
# UDP server? Use the same setting as
# on the server.
;proto tcp
proto udp

# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
;remote my-server-2 1194

# Choose a random host from the remote
# list for load-balancing. Otherwise
# try hosts in the order specified.
;remote-random

# Keep trying indefinitely to resolve the
# host name of the OpenVPN server. Very useful
# on machines which are not permanently connected
# to the internet such as laptops.
resolv-retry infinite

# Most clients don't need to bind to
# a specific local port number.
nobind

# Downgrade privileges after initialization (non-Windows only)
;user nobody
;group nogroup

# Try to preserve some state across restarts.
persist-key
persist-tun

# If you are connecting through an
# HTTP proxy to reach the actual OpenVPN
# server, put the proxy server/IP and
# port number here. See the man page
# if your proxy server requires
# authentication.
;http-proxy-retry # retry on connection failures
;http-proxy [proxy server] [proxy port #]

# Wireless networks often produce a lot
# of duplicate packets. Set this flag
# to silence duplicate packet warnings.
;mute-replay-warnings

# SSL/TLS parms.
# See the server config file for more
# description. It's best to use
# a separate .crt/.key file pair
# for each client. A single ca
# file can be used for all clients.
ca ca.crt
cert client1.crt
key client1.key

# Verify server certificate by checking
# that the certicate has the nsCertType
# field set to "server". This is an
# important precaution to protect against
# a potential attack discussed here:
# http://openvpn.net/howto.html#mitm
#
# To use this feature, you will need to generate
# your server certificates with the nsCertType
# field set to "server". The build-key-server
# script in the easy-rsa folder will do this.
ns-cert-type server

# If a tls-auth key is used on the server
# then every client must also have the key.
tls-auth ta.key 1

# Select a cryptographic cipher.
# If the cipher option is used on the server
# then you must also specify it here.
cipher AES-256-CBC
auth SHA512

# Enable compression on the VPN link.
# Don't enable this unless it is also
# enabled in the server config file.
comp-lzo adaptive

# Set log file verbosity.
verb 3

# Silence repeating messages
mute 20

# Log
log-append client.log
status status.log
```

### Linux

Расширение файла `.config`. Логи хранятся в `/var/log/`. Пользователь от которого запускается клиент openvpn - nobody.
```bash
##############################################
# Sample client-side OpenVPN 2.0 config file #
# for connecting to multi-client server.    #
# This configuration can be used by multiple #
# clients, however each client should have  #
# its own cert and key files.               #
# On Windows, you might want to rename this #
# file so it has a .ovpn extension          #
##############################################

# Specify that we are a client and that we
# will be pulling certain config file directives
# from the server.
client

# Use the same setting as you are using on
# the server.
# On most systems, the VPN will not function
# unless you partially or fully disable
# the firewall for the TUN/TAP interface.
;dev tap
dev tun

# Windows needs the TAP-Win32 adapter name
# from the Network Connections panel
# if you have more than one. On XP SP2,
# you may need to disable the firewall
# for the TAP adapter.
;dev-node MyTap

# Are we connecting to a TCP or
# UDP server? Use the same setting as
# on the server.
;proto tcp
proto udp

# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
;remote my-server-2 1194

# Choose a random host from the remote
# list for load-balancing. Otherwise
# try hosts in the order specified.
;remote-random

# Keep trying indefinitely to resolve the
# host name of the OpenVPN server. Very useful
# on machines which are not permanently connected
# to the internet such as laptops.
resolv-retry infinite

# Most clients don't need to bind to
# a specific local port number.
nobind

# Downgrade privileges after initialization (non-Windows only)
user nobody
group nogroup

# Try to preserve some state across restarts.
persist-key
persist-tun

# If you are connecting through an
# HTTP proxy to reach the actual OpenVPN
# server, put the proxy server/IP and
# port number here. See the man page
# if your proxy server requires
# authentication.
;http-proxy-retry # retry on connection failures
;http-proxy [proxy server] [proxy port #]

# Wireless networks often produce a lot
# of duplicate packets. Set this flag
# to silence duplicate packet warnings.
;mute-replay-warnings

# SSL/TLS parms.
# See the server config file for more
# description. It's best to use
# a separate .crt/.key file pair
# for each client. A single ca
# file can be used for all clients.
ca ca.crt
cert client1.crt
key client1.key

# Verify server certificate by checking
# that the certicate has the nsCertType
# field set to "server". This is an
# important precaution to protect against
# a potential attack discussed here:
# http://openvpn.net/howto.html#mitm
#
# To use this feature, you will need to generate
# your server certificates with the nsCertType
# field set to "server". The build-key-server
# script in the easy-rsa folder will do this.
ns-cert-type server

# If a tls-auth key is used on the server
# then every client must also have the key.
tls-auth ta.key 1

# Select a cryptographic cipher.
# If the cipher option is used on the server
# then you must also specify it here.
cipher AES-256-CBC
auth SHA512

# Enable compression on the VPN link.
# Don't enable this unless it is also
# enabled in the server config file.
comp-lzo adaptive

# Set log file verbosity.
verb 3

# Silence repeating messages
mute 20

# Log
log-append /var/log/client.log
status /var/log/status.log
```

### PS

Как сделать привязку IP к сертификату клиента написано [тут](https://nesterof.com/blog/2017/09/17/static-ip-openvpn/). Как сделать сертификат для клиента в укороченом виде [тут](https://nesterof.com/blog/2017/09/17/create-client-cert-openvpn/)