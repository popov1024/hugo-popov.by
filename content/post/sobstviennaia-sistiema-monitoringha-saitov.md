---
title: "Собственная система мониторинга сайтов"
date: 2018-07-15
draft: false
description: "Мониториинг сайтов с помощью Monit"
tags: [
	"docker",
  "monit",
  digitalocean
]
---

Те, кто занимается поддержкой сайтов рано или поздно задумывается как мониторить их доступность. Можно пойти по пути покупки услуг (или пользоваться бесплатными) соответствущих сервисов. Благо их много.

Но тут всегда сталкиваемся с несколькими проблемами: периодичность опроса и гибкость настройки.

Другой вариант поднять свой сервис мониторинга, о чем и пойдет далее речь.

Для наших целей будем использовать следующие сервисы и технологии:

- [DigitalOcean (referal link)](https://m.do.co/t/18905df36c8e)
- [Monit](https://mmonit.com/monit/)
- [Docker](https://docker.com/)

### DigitalOcean

Тут все просто. Регистрируемся и создаем дроплет. Можно воспользоваться моей реферальной ссылкой [DigitalOcean (referal link)](https://m.do.co/t/18905df36c8e).
Для наших целей подойдет с минимальными характеристиками (1Gb / 1CPU). Можно воспользоваться уже специльно подготовленным образом `One-click apps` с установленным `Docker`.

Место расположение дроплета - тут по своему вкусу. В идеале в той стране, где нужно контролировать доступность. В хорошем варианте можно поднять несколько однотипных дроплетов в различных датацентрах.

### Monit

Monit - это легковесная система мониторинга. Как одним из видов мониторинга выступает проверка подключение к внешним сервисам по HTTP(S).

Все настройки произвлдятся в виде [конфиг файлов](https://github.com/popov1024/monit-alpine-docker/blob/master/monitrc) с использованием специализированного синтаксиса.

``` shell
check host example.com with address example.com
    alert monit@example.com
    if failed url http://example.com/ with timeout 5 seconds for 3 cycles then alert
```

### Docker

Можно установить сам `monit` и на этом остановиться. Но удобнее для переносимости и утилизации системных ресурсов (вдруг мы еще что-то захотим запускать одновременно) использовать `Docker`. Создаем `docker-compose.yml`:

``` shell
version: '2'
services:
  monit:
    image: monit-alpine
    build:
      context: .
      dockerfile: Dockerfile
    restart: always
    hostname: monit
    logging:
      driver: json-file
      options:
        max-size: "10m"
        max-file: "5"
    volumes:
      - './monit/monit.d/:/etc/monit.d/'
      - './monit/monitrc:/etc/monitrc'
    restart: always
```

и создаем в этой же директории `Dockerfile`

``` shell
FROM alpine:latest

RUN apk --no-cache add monit

VOLUME /etc/monit.d/

EXPOSE 2812

CMD ["monit", "-I", "-B"]
```

Запускаем стандартно:

``` shell
docker-compose up -d
```
