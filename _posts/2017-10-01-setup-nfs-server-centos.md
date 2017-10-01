---
title: Как поднять сервер NFS в CentOS 6.8
layout: post
categories: NFS
tags: nfs centos linux
---

> OC: CentOS 6.8(Final)

```
# yum install nfs-utils nfs-utils-lib
# service rpcbind start
# service nfs start
# chkconfig nfs on && chkconfig rpcbind on
Starting NFS services: [ OK ]
Starting NFS mountd: [ OK ]
Starting NFS daemon: [ OK ]
Starting RPC idmapd: [ OK ]
```
Редактируем конфигурационный файл NFS сервера `# joe /etc/exports`

```bash
/mnt/share 192.168.1.0/24(rw,no_root_squash)
```
* /mnt/share - путь к папке, для которой раздается доступ.
* 192.168.1.0/24 - подсеть для который будет доступен расшаренный ресурс (или ip клиента).
* (rw, no_root_squash) - опции.

### Опции

* rw –чтение запись(или ro - только чтение).
* no_root_squash –по умолчанию у пользователя root на клиентской машине нет доступа к разделяемой директории сервера. Этой опция снимает это ограничение. В целях безопасности этого лучше не делать.
* nohide - NFS автоматически не показывает нелокальные ресурсы (например примонтированые с помощью mount –bind), эта опция включает отображение таких ресурсов;
* sync - синхронный режим доступа (обратное значение - async). Sync - указывает, что сервер отвечает на запросы только после записи на диск изменений, выполненных этими запросами. Опция async указывает серверу не ждать записи информации на диск, что повышает производительность, но понижает надежность, т.к. при отказе оборудования или сети возможна потеря данных.
* noaccess - запрещает доступ к указанной директории. Полезна когда у пользователей сети  есть доступ к  директории, и нужно разграничить доступ в поддиректории некоторым пользователям.
* all_squash - подразумевает, что подключения будут выполнятся от анонимного пользователя.
* subtree_check (no_subtree_check) - иногда приходится экспортировать только часть раздела. При этом сервер NFS выполняет дополнительную проверку обращений клиентов, чтобы убедиться в том, что они предпринимают попытку доступа к файлам, находящимся в соответствующих подкаталогах. Такой контроль поддерева (subtree checks) замедляет взаимодействие с клиентами, но если от него отказаться, возникнут проблемы с безопасностью системы. Отменить контроль поддерева можно с помощью опции no_subtree_check. Опция subtree_check, включающая такой контроль, предполагается по умолчанию. Контроль поддерева можно не выполнять в том случае, если экспортируемый каталог совпадает с разделом диска.
* anonuid=1000 - привязывает анонимного пользователя к «местному» пользователю.
* anongid=1000 - привязывает анонимного пользователя к группе «местного» пользователя.

Еще опции монтирования, указывается что работают для fstab, mount и autofs (These options can be used with manual mount commands, /etc/fstab settings, and autofs.) Возможно работают и для /etc/exports.

* fsid=num — Forces the file handle and file attributes settings on the wire to be num, instead of a number derived from the major and minor number of the block device on the mounted file system. The value 0 has special meaning when used with NFSv4. NFSv4 has a concept of a root of the overall exported file system. The export point exported with fsid=0 is used as this root.
* hard or soft — Specifies whether the program using a file via an NFS connection should stop and wait (hard) for the server to come back online, if the host serving the exported file system is unavailable, or if it should report an error (soft).
If hard is specified, the user cannot terminate the process waiting for the NFS communication to resume unless the intr option is also specified.
If soft is specified, the user can set an additional timeo=<value> option, where <value> specifies the number of seconds to pass before the error is reported.

**Note**

**Using soft mounts is not recommended as they can generate I/O errors in very congested networks or when using a very busy server.**

* intr — Allows NFS requests to be interrupted if the server goes down or cannot be reached.
* nfsvers=2 or nfsvers=3 — Specifies which version of the NFS protocol to use. This is useful for hosts that run multiple NFS servers. If no version is specified, NFS uses the highest supported version by the kernel and mount command. This option is not supported with NFSv4 and should not be used.
* noacl — Turns off all ACL processing. This may be needed when interfacing with older versions of Red Hat Enterprise Linux, Red Hat Linux, or Solaris, since the most recent ACL technology is not compatible with older systems.
* nolock — Disables file locking. This setting is occasionally required when connecting to older NFS servers.
* noexec — Prevents execution of binaries on mounted file systems. This is useful if the system is mounting a non-Linux file system via NFS containing incompatible binaries.
* nosuid — Disables set-user-identifier or set-group-identifier bits. This prevents remote users from gaining higher privileges by running a setuid program.
* port=num — Specifies the numeric value of the NFS server port. If num is 0 (the default), then mount queries the remote host's portmapper for the port number to use. If the remote host's NFS daemon is not registered with its portmapper, the standard NFS port number of TCP 2049 is used instead.
* rsize=num and wsize=num — These settings speed up NFS communication for reads (rsize) and writes (wsize) by setting a larger data block size, in bytes, to be transferred at one time. Be careful when changing these values; some older Linux kernels and network cards do not work well with larger block sizes. For NFSv2 or NFSv3, the default values for both parameters is set to 8192. For NFSv4, the default values for both parameters is set to 32768.
* sec=mode — Specifies the type of security to utilize when authenticating an NFS connection.
* sec=sys is the default setting, which uses local UNIX UIDs and GIDs by means of AUTH_SYS to authenticate NFS operations.
* sec=krb5 uses Kerberos V5 instead of local UNIX UIDs and GIDs to authenticate users.
* sec=krb5i uses Kerberos V5 for user authentication and performs integrity checking of NFS operations using secure checksums to prevent data tampering.
* sec=krb5p uses Kerberos V5 for user authentication, integrity checking, and encrypts NFS traffic to prevent traffic sniffing. This is the most secure setting, but it also has the most performance overhead involved.
* tcp — Specifies for the NFS mount to use the TCP protocol.
* udp — Specifies for the NFS mount to use the UDP protocol.

Применяем изменения и проверяем.
```
# exportfs -a
# exportfs -v
/mnt/share 192.168.1.0/24(rw,wdelay,root_squash,no_subtree_check,sec=sys,rw,root_squash,no_all_squash)
```