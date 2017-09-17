---
title: Автозапуск виртуальных машин
layout: post
categories: XenServer
tags: xenserver
---

> OC: XEN 7

Командой `xe pool-list` смотрим UUID пула, для которого мы хотим включить Auto Start и копируем **uuid(RO)**.
Разрешаем автозапуск виртуальных машин:

```
# xe pool-param-set uuid=(insert UUID(RO)) other-config:auto_poweron=true
```

Командой `# xe vm-list` смотрим список виртуальных машин и копируем **uuid(RO)** нужной машины.
Включаем автостарт для нужной машины:

```
# xe pool-param-set uuid=(insert UUID(RO)) other-config:auto_poweron=true
```

## Автозапуск виртуальных машин с задержкой

> OC: XEN 6.5

Возможно работает и на XEN 7 я не проверял.

* Командой `# xe vm-list` узнаем uuid ( RO) виртуальной машины

```
uuid ( RO) : 2a3f702d-6fb0-0d8c-a670-e124fd7cb6e8
name-label ( RW): debian.client
power-state ( RO): halted
```

* Прописываем в `/etc/rc.local` виртуальные машины которые нужно запускать автоматически.

Параметр `sleep 20` - задержка 20 секунд (необязательный).

```
sleep 20
xe vm-start uuid=2a3f702d-6fb0-0d8c-a670-e124fd7cb6e8
```

Если VM несколько так:

```
# First VM
sleep 20
xe vm-start uuid=2a3f702d-6fb0-0d8c-a670-e124fd7cb6e8
# Second VM
sleep 40
xe vm-start uuid=3b4a321d-7dw1-1a9c-a670-e1r23sg5df61
```

Можно также использовать `name-label` например:

```
# xe vm-list params | grep name-label
name-label ( RW): centos.server
name-label ( RW): debian.client
```

В `/etc/rc.local` следует писать тогда так:

```
# First VM
sleep 20
xe vm-start vm=centos.server
# Second VM
sleep 40
xe vm-start vm=debian.client
```

Назначать по именам неудобно, придется каждый раз ~~править исходники~~ переписывать конфигурацию при изменении имени VM.