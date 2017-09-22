---
title: Экспорт репозитария Mercurial в Git
layout: post
categories: VCS
tags: mercurial git debian linux
---

> OC: Debian 8 Jessie

Взято [тут](https://hedonismbot.wordpress.com/2008/10/16/hg-fast-export-convert-mercurial-repositories-to-git-repositories/). 

Клонируем скрипт экспорта `$ git clone git://repo.or.cz/fast-export.git`

Создаем пустой Git репозитарий

```
$ mkdir GitRepo
$ cd GitRepo
$ git init
Initialized empty Git repository in ~/ExportToGit/GitRepo /.git/
```

Начинаем экспорт и получаем ошибку (питон 2.7.1 уже установлен). Ошибка кстати будет такая же если запустить .sh версию скрипта.

```
$ ~/ExportToGit/fast-export/hg-fast-export.py -r ~/ExportToGit/HgRepo
Traceback (most recent call last):
  File "/home/nesterov/ExportToGit/fast-export/hg-fast-export.py", line 6, in <module>
    from mercurial import node
ImportError: No module named mercurial
```

Ставим нехватающий модуль
```
# apt-get install python-pip
# pip install mercurial
```

И повторяем `$ ~/ExportToGit/fast-export/hg-fast-export.sh -r ~/ExportToGit/HgRepo`

Если все хорошо, увидим отчет о выполнении в таком виде:

```
nesterov_160718_sync_test_refact: Exporting simple delta revision 632/649 with 0/1/0 added/changed/removed files
nesterov_160718_sync_test_refact: Exporting simple delta revision 633/649 with 0/1/0 added/changed/removed files
nesterov_160718_sync_test_refact: Exporting simple delta revision 634/649 with 0/8/1 added/changed/removed files
nesterov_160718_sync_test_refact: Exporting simple delta revision 635/649 with 4/5/1 added/changed/removed files
nesterov_160718_sync_test_refact: Exporting simple delta revision 636/649 with 0/2/0 added/changed/removed files
nesterov_160718_sync_test_refact: Exporting simple delta revision 637/649 with 0/7/1 added/changed/removed files
nesterov_161019_fix_ms_cut_sheet: Exporting simple delta revision 638/649 with 0/0/0 added/changed/removed files
nesterov_161019_fix_ms_cut_sheet: Exporting simple delta revision 639/649 with 0/1/0 added/changed/removed files
nesterov_161021_update_barcode_scanner: Exporting simple delta revision 640/649 with 0/3/0 added/changed/removed files
nesterov_161021_update_barcode_scanner: Exporting simple delta revision 641/649 with 0/1/0 added/changed/removed files
nesterov_161021_update_barcode_scanner: Exporting simple delta revision 642/649 with 0/1/0 added/changed/removed files
nesterov_161021_update_barcode_scanner: Exporting simple delta revision 643/649 with 0/1/0 added/changed/removed files
nesterov_161021_update_barcode_scanner: Exporting simple delta revision 644/649 with 0/0/0 added/changed/removed files
master: Exporting thorough delta revision 645/649 with 0/3/0 added/changed/removed files
master: Exporting simple delta revision 646/649 with 1/0/0 added/changed/removed files
master: Exporting thorough delta revision 647/649 with 1/3/0 added/changed/removed files
master: Exporting simple delta revision 648/649 with 0/1/0 added/changed/removed files
master: Exporting simple delta revision 649/649 with 0/0/1 added/changed/removed files
```

А в конце статистику экспорта

```
Issued 649 commands
git-fast-import statistics:
---------------------------------------------------------------------
Alloc'd objects:      15000
Total objects:        11552 (2994 duplicates)
      blobs  :         6663 (2587 duplicates       3639 deltas of       6598 attempts)
      trees  :         4240 (407 duplicates       3450 deltas of       3951 attempts)
      commits:          649 (0 duplicates          0 deltas of          0 attempts)
      tags   :            0 (0 duplicates          0 deltas of          0 attempts)
Total branches:          16 (19 loads)
      marks:           1024 (649 unique)
      atoms:           1600
Memory total:          2876 KiB
       pools:          2173 KiB
     objects:           703 KiB
---------------------------------------------------------------------
pack_report: getpagesize()            =       4096
pack_report: core.packedGitWindowSize = 1073741824
pack_report: core.packedGitLimit      = 8589934592
pack_report: pack_used_ctr            =       5326
pack_report: pack_mmap_calls          =       2434
pack_report: pack_open_windows        =          1 /          1
pack_report: pack_mapped              =   59922644 /   59922644
---------------------------------------------------------------------
```

Самое главное - в конце сделать `$ git checkout HEAD`. Иначе все взорвется. 

P. S. Виндовозный клиент Git Bash просил еще сделать  `git config core.ignorecase false`, а дебиановский нет. Почему так происходит хуй знает.