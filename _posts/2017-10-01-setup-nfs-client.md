---
title: Как поднять клиент NFS
layout: post
categories: NFS
tags: nfs centos linux
---

> OC: Debian 8 Jessie

```
# apt-get install nfs-client
```

В `/etc/idmapd.conf` раскоментировать (там будет hostname машины)
```
Domain = test.localdomain
```

Потом
```
# tests mount
# mount -t nfs 192.168.1.1:/mnt/nfs /mnt
```