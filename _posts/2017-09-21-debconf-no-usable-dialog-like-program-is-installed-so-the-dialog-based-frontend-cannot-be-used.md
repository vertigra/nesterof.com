---
layout: post
categories: Errors
tags: errors debian linux
title: 'debconf: (No usable dialog-like program is installed, so the dialog based
  frontend cannot be used)'
---

> OC: Debian 8 Jessie


### Симптомы

```
# apt-get install some_program
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 76.)
debconf: falling back to frontend: Readline
```

### Решение

```
apt-get install dialog
```

или так:

```
apt-get install whiptail
```