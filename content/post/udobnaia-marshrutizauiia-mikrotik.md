---
title: "Удобная маршрутизауия mikrotik"
date: 2018-07-15
draft: false
---

Необходимо маршрутизировать несколько ip или подсетей по одному правилу.

Решение
Вариант "в лоб" прописывать все это в таблице маршрутизации. Все хорошо работает, когда подсетей несколько, а когда их достигает десяток, да и еще вперемешку на разные шлюзы...

`RouterOS` поддерживает работу со списками и маркировку пакетов.

И так, создаем список `work`:

``` sh
/ip firewall address-list
add address=10.10.0.0/16 list=work
add address=192.168.1.0/24 list=work
```

Делаем пометку `workt` пакетов согласно данного списка work:

``` sh
/ip firewall mangle
add action=mark-routing chain=prerouting dst-address-list=work new-routing-mark=workt
```

Собственно маршрутизация, согласно пометки `workt`:

``` sh
/ip route
add distance=1 gateway=vpn-work routing-mark=workt
```

Теперь достаточно в список `work` внести новый `ip` или подсеть, и он будет маршрутизироваться согласно правилам для этого листа.
