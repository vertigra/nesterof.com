---
title: 'wget: The certificate of ... is not trusted'
layout: post
categories: Errors
tags: errors debian linux
---

> OC: Debian 8 Jessie

### Симптомы

```
# wget https://www.dropbox.com/s/a1b2c3d4ef5gh6/example.docx?dl=1
ERROR: The certificate of `www.dropbox.com' is not trusted.
ERROR: The certificate of `www.dropbox.com' hasn't got a known issuer.
```

### Решение

```
# apt-get install ca-certificates
```

[Ветка на StackOverflow](http://stackoverflow.com/questions/9224298/how-do-i-fix-certificate-errors-when-running-wget-on-an-https-url-in-cygwin)