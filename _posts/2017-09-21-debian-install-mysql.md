---
title: "Как установить MySQL сервер"
layout: post
categories: MySql
tags: mysql debian linux
---

> OC: Debian 8 Jessie

Взято [тут](https://www.linode.com/docs/databases/mysql/how-to-install-mysql-on-debian-8). Версия MySQL 5.5.

```
# apt-get update
# apt-get upgrade
# apt-get install mysql-server
```

В процессе установки будет запрошен пароль для администратора БД (root).
Слушает по умолчанию 127.0.0.1.

---

**Note**

> Allowing unrestricted access to MySQL on a public IP not advised, but you may change the address it listens on by modifying the bind-address parameter in /etc/my.cnf. If you decide to bind MySQL to your public IP, you should implement firewall rules that only allow connections from specific IP addresses.

---

Запускаем скрипт установки безопасности. Подробнее о нем [тут](https://dev.mysql.com/doc/refman/5.7/en/mysql-secure-installation.html)
```bash
# mysql_secure_installation
```

отвечаем так:
```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MySQL
SERVERS IN PRODUCTION USE! PLEASE READ EACH STEP CAREFULLY!


In order to log into MySQL to secure it, we'll need the current
password for the root user. If you've just installed MySQL, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none):
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MySQL
root user without the proper authorisation.

You already have a root password set, so you can safely answer 'n'.

Change the root password? [Y/n] n
... skipping.

By default, a MySQL installation has an anonymous user, allowing anyone
to log into MySQL without having to have a user account created for
them. This is intended only for testing, and to make the installation
go a bit smoother. You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
... Success!

Normally, root should only be allowed to connect from 'localhost'. This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
... Success!

By default, MySQL comes with a database named 'test' that anyone can
access. This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
- Dropping test database...
ERROR 1008 (HY000) at line 1: Can't drop database 'test'; database doesn't exist
... Failed! Not critical, keep moving...
- Removing privileges on test database...
... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
... Success!

Cleaning up...



All done! If you've completed all of the above steps, your MySQL
installation should now be secure.

Thanks for using MySQL!
```