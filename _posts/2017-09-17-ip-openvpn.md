---
title: Как сделать привязку IP к сертификату клиента OpenVPN
layout: post
categories: OpenVPN
tags: linux, openvpn
---

Для привязки ip к сертификату клиента, необходимо:

* Указать в конфигурации сервера директорию хранения конфигурационных файлов клиента. По умолчанию это /etc/openvpn/ccd,  в конфигурационном файле за это отвечает параметр `client-config-dir ccd`

```
# To assign specific IP addresses to specific
# clients or if a connecting client has a private
# subnet behind it that should also have VPN access,
# use the subdirectory "ccd" for client-specific
# configuration files (see man page for more info).
# EXAMPLE: Suppose the client
# having the certificate common name "Thelonious"
# also has a small subnet behind his connecting
# machine, such as 192.168.40.128/255.255.255.248.
# First, uncomment out these lines:
;client-config-dir ccd
;route 192.168.40.128 255.255.255.248
# Then create a file ccd/Thelonious with this line:
# iroute 192.168.40.128 255.255.255.248
# This will allow Thelonious' private subnet to
# access the VPN. This example will only work
# if you are routing, not bridging, i.e. you are
# using "dev tun" and "server" directives.
# EXAMPLE: Suppose you want to give
# Thelonious a fixed VPN IP address of 10.9.0.1.
# First uncomment out these lines:
client-config-dir ccd
```

* в директории ccd создать файл с именем указанным в Common Name сертификата
* указать в созданном файле конфигурацию в формате `ifconfig-push ip-клиента ip-шлюза`, ip клиента и шлюза должны соответствовать пулу адресов OpenVPN сети, который можно срисовать в параметре `server 192.168.1.0 255.255.255.0`

```
# Configure server mode and supply a VPN subnet
# for OpenVPN to draw client addresses from.
# The server will take 10.8.0.1 for itself,
# the rest will be made available to clients.
# Each client will be able to reach the server
# on 10.8.0.1. Comment this line out if you are
# ethernet bridging. See the man page for more info.
server 192.168.1.0 255.255.255.0
```

* из-за разделения OpenVPN сети на подсети из четырех адресов: network, IP, gateway, broadcast, ip клиента и шлюза должны быть сконфигурированы следующим образом:

```
[ip клиента, ip шлюза]
[ 1, 2] [ 5, 6] [ 9, 10] [ 13, 14] [ 17, 18]
[ 21, 22] [ 25, 26] [ 29, 30] [ 33, 34] [ 37, 38]
[ 41, 42] [ 45, 46] [ 49, 50] [ 53, 54] [ 57, 58]
[ 61, 62] [ 65, 66] [ 69, 70] [ 73, 74] [ 77, 78]
[ 81, 82] [ 85, 86] [ 89, 90] [ 93, 94] [ 97, 98]
[101,102] [105,106] [109,110] [113,114] [117,118]
[121,122] [125,126] [129,130] [133,134] [137,138]
[141,142] [145,146] [149,150] [153,154] [157,158]
[161,162] [165,166] [169,170] [173,174] [177,178]
[181,182] [185,186] [189,190] [193,194] [197,198]
[201,202] [205,206] [209,210] [213,214] [217,218]
[221,222] [225,226] [229,230] [233,234] [237,238]
[241,242] [245,246] [249,250] [253,254]
```

Пример привязки ip 192.168.1.41 к сертификату с CN examplename

```
# echo 'ifconfig-push 192.168.1.41 192.168.1.42' > /etc/openvpn/ccd/examplename
```