---
title: 'Starting NFS statd: [FAILED]'
layout: post
categories: Errors
tags: errors centos linux
---

> OC: CentOS 6.8 (Final)

### Симптомы

При загрузке система виснет на этапе `Starting NFS statd:` после некоторого времени продолжает загрузку, при этом статус запуска NFS демона `[FAILED]` (`Starting NFS statd: [FAILED]`).

### Решение

Дождаться загрузки ситемы, и добавить службы NFS в автозагрузку 
```
# chkconfig nfs on && chkconfig rpcbind on
```