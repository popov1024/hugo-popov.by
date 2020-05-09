---
title: "Востановление флешки Corsair Survivor"
date: 2011-08-29T00:00:00+03:00
draft: false
---

Казалось бы, что флешки `Corsair Survivor` неубиваемы...

Однако моя в один прекрасный момент умерла. Причем контроллер опридилялась в системе, но без диска.
Последовательность востановления:
1 Определяем `VID` и `PID` контроллера. Можно с помощью программы `Chip Cenius 3.00`

``` none
Device Name: +[E:]+Запоминающее устройство для USB(Ut165 USB Flash Disk USB Device)

PnP Device ID: VID = 1307 PID = 0165
Serial Number: 00000000000004
Revision: 0.00

Device Type: Standard USB device - USB2.0 High-Speed

Chip Vendor: USBest
Chip Part-Number: UT165

Product Vendor: (N/A)
Product Model: (N/A)

Tools on Web: ?http://bbs.mydigit.cn/read.php?tid=148892 
```

2 На сайте www.flashboot.ru/iflash.html ищем утилиты для работы с контроллером.
Помогло мне 2-е утилиты.
`Super Stick Recovery Tool V1.0.2.19` - Нет никаких настроек и после нее скорость записи упала в два раза.
`UT165-MPTool_V1.65.30.0` - Можно настроить практически все и посмотреть практически все. Нашла у меня 6-ть "плохих" блоков.

В результате - полностью работоспособная флешка.