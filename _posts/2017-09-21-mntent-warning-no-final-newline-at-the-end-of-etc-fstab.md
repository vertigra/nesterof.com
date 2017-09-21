---
title: "[mntent]: warning: no final newline at the end of /etc/fstab"
layout: post
categories: Errors
tags: errors centos linux
---

> OC: CentOs 6.8 (Final)

### Симптомы

При загрузке системы появляется сообщение `[mntent]: warning: no final newline at the end of /etc/fstab` при этом файловые системы нормально монтируются.

### Решение

Добавить `newline` в `/etc/fstab` (перейти в конец последней строки файла /etc/fstab и нажать `enter`).