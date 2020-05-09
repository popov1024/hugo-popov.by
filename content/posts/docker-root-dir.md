---
title: "Docker Root Dir"
date: 2018-04-30
draft: false
---

По умолчанию `Docker` хранит свое окружение (образа, контейнеры и т.д.) в `/var/lib/docker`. Однако часто может возникать проблема нехватки места из-за большого размера контейнеров при хранении в `/var` на корневом томе.

В примере показн вариант переноса корневой дирректории `docker` на смонтированный том `/u01` :

``` shell
sudo systemctl stop docker
sudo cp -rp /var/lib/docker/ /u01/
sudo mv /var/lib/docker /var/lib/docker.old
sudo ln -s /u01/docker/ /var/lib/
sudo systemctl daemon-reload
sudo systemctl start docker
```

Для проверки выполним `docker info` и проверим значением параметра `Docker Root Dir:`.
