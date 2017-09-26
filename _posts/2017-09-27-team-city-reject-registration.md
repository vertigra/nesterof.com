---
title: Как запретить регистрацию пользвателей в Team City 10.0.2
layout: post
categories: TeamCity
tags: teamcity java debian linux
---

>OC: Debian 8 Jessie

Чтобы запретить регистрацию новых пользователей (Register a new user account на странице авторизации TeamCity) нужно:

* Авторизоваться с правами администратора TeamCity
* Перейти на страницу Administration > Authentication
* В настройках модуля аутентификации (Credentials authentication modules > Edit) снять галку Allow user registration from the login page.