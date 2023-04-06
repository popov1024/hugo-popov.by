---
title: "Настройка собственного VPN сервера на базе Wireguard"
date: 2023-04-06T09:59:13+03:00
draft: false
tags: [
    "docker", "vpn", "mikrotik"
]
---

## Конфигурирование `Wireguard` сервера

Первым делом требуется создать VPS сервер в каком-либо облачном провайдере. Я использую [DigitalOcean](https://m.do.co/c/18905df36c8e) и получаю бонусные баллы за каждого нового пользователя, который регистрируется по моей реферальной ссылке.

[![DigitalOcean Referral Badge](https://web-platforms.sfo2.cdn.digitaloceanspaces.com/WWW/Badge%201.svg)](https://www.digitalocean.com/?refcode=18905df36c8e&utm_campaign=Referral_Invite&utm_medium=Referral_Program&utm_source=badge)

Вы можете создать VPS в люом другом облачном провайдере (AWS, Google Cloud, Azure, Yandex Cloud, etc.).

Требуется создать дроплет с установленным Docker. 
Можно использовать образ с предустановленным докером, либо установить докер самостоятельно (пример для [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04-ru)).

После создания дроплета необходимо установить и настроить `Wireguard` сервер.

### Установка `Wireguard` сервера

Для установки `Wireguard` сервера используется официальный образ [linuxserver/wireguard](https://hub.docker.com/r/linuxserver/wireguard).

``` shell
docker run -d \
  --name=wireguard \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  -e PUID=1000 \
  -e PGID=1000 \
  -e TZ=Etc/UTC \
  -e SERVERURL=auto \
  -e SERVERPORT=51820 \
  -e PEERS=4 \
  -e PEERDNS=auto \
  -e INTERNAL_SUBNET=10.13.13.0 \
  -e ALLOWEDIPS=0.0.0.0/0 \
  -p 51820:51820/udp \
  -v ~/wireguard/config:/config \
  -v /lib/modules:/lib/modules `#optional` \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --restart unless-stopped \
  lscr.io/linuxserver/wireguard
```

Или можно использовать `docker-compose`:
```
version: "2.1"
services:
  wireguard:
    image: lscr.io/linuxserver/wireguard
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - SERVERURL=auto #optional
      - SERVERPORT=51820 #optional
      - PEERS=4 #optional
      - PEERDNS=auto #optional
      - INTERNAL_SUBNET=10.13.13.0 #optional
      - ALLOWEDIPS=0.0.0.0/0 #optional
    volumes:
      - ~/wireguard/config:/config
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
```
После успешного запуска контенера можно проверить статус `Wireguard` сервера:
``` shell
docker exec -it wireguard wg
```

## Настройка клиента `Wireguard`

В нашем примере сервра создано 4 пира, поэтому можно подключиться к любому из них. Но желательно подключать одного клиента к одному пиру.

Для подключения к `Wireguard` серверу необходимо установить клиентское приложение `Wireguard` на устройство, с которого будет осуществляться подключение. Скачать `Wireguard` можно с [официального сайта](https://www.wireguard.com/install/).

Для мобильных клиентов доступны приложения для [Android](https://play.google.com/store/apps/details?id=com.wireguard.android) и [iOS](https://apps.apple.com/us/app/wireguard/id1441195209).

После установки клиента необходимо импортировать конфигурационный файл `Wireguard` сервера. Для этого необходимо нажать на кнопку `+` в правом верхнем углу экрана и выбрать пункт `Import from QR code` или `Import from file`.

QR код можно сгенерировать с помощью команды (пример для пира с номером 1):

``` sh
docker exec -it wireguard /app/show-peer 1
```

Конфигурация пиров расположена в директории `~/wireguard/config/`, где в кажоый директории `peer*` находится конфигурация пира напрмиер, `peer1.conf`.


После импорта конфигурационного файла необходимо нажать на кнопку `Activate` и подключиться к `Wireguard` серверу.


## Настройка `Mikrotik` для подключения к `Wireguard` серверу.

Я использую Mikrotik для маршрутизации трафика в локальной сети.

В моем случае я сконфигурировал маршрутизация не всего трафига через `Wireguard` сервер, а только трафик к определенным ресурсам.

```
/interface wireguard add listen-port=51820 mtu=1420 name=wireguard1
/interface wireguard peers add allowed-address=0.0.0.0/0 endpoint-address=<ENDPOINT_ADDRESS> endpoint-port=<ENDPOINT_PORT> interface=wireguard1 public-key="<PUBLICKEY>"
/routing table add disabled=no fib name=vpnt
/ip firewall address-list add address=whatismyipaddress.com list=vpn
/ip address add address=<PEERIP> interface=wireguard1 network=10.13.0.0
/ip firewall mangle add action=mark-routing chain=prerouting comment=vpn dst-address-list=vpn new-routing-mark=vpnt passthrough=yes
/ip firewall nat add action=masquerade chain=srcnat out-interface=wireguard1
/ip route add comment=vpn disabled=no distance=2 dst-address=0.0.0.0/0 gateway=wireguard1 pref-src="" routing-table=vpnt scope=10 suppress-hw-offload=no
```

<ENDPOINT_ADDRESS>, <ENDPOINT_PORT>, <PUBLICKEY>, <PEERIP> - необходимо заменить на свои значения из конфигурации пира.

В моем случае я использую `whatismyipaddress.com` для определения моего внешнего IP адреса. Для этого я добавил этот ресурс в список адресов `ip firewall address-list`. Так же я добавил маршрут по умолчанию через `wireguard1` с меткой `vpnt` и добавил правило для маршрутизации трафика с меткой `vpnt` через `wireguard1`. Таким образом, трафик к ресурсам из списка `vpn` будет маршрутизироваться через `wireguard1`.
