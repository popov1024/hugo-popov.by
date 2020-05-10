---
title: Mikrotik на страже рекламы
date: 2018-11-13
draft: false
description: "Mikrotik - удаление рекламы"
tags: [
    "mikrotik",
    "ad"
]
---

Блокировка рекламы на Mikrotik можно настроить с помощью переоидиечкого запуска скрипта, который для url рекламного контента  подменяет ip на 127.0.0.1.

* Скачиваем текст скрипта <https://www.micu.eu/adblock/adblock.php> и сохраняем как adblock.rsc.
* Загружаем файл в Mikrotik
* Создаем script

``` shell
add dont-require-permissions=no name=adblock owner=admin policy=read,write,policy,test source=":log info \1CAdblock: apply start\1D\n/ import file-name=adblock.rsc\n:log info \1CAdblock: apply finished\1D"
```

* Создаем расписание запуска

``` shell
add interval=10m name=adblock on-event="/system script run adblock" policy=read,write,policy,test start-date=nov/13/2018 start-time=18:15:54
```
