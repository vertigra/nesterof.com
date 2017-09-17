---
title: Как установить пакет на Xen Server
layout: post
categories: XenServer
tags: xenserver
---

> OC: XEN 6.5

Очень просто:

```
# yum install --enablerepo=base package-name
```

Например, чтобы установить usbutils

```
# yum install --enablerepo=base usbutils
```

Должно работать и в XEN 7.