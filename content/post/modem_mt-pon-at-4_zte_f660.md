---
title: Модем MT-PON-AT-4 (ZTE F660)
date: 2018-12-04
draft: false
---

PON модем `MT-PON-AT-4` (`ZTE F660`) имеет возможность доступа по telnet.
![Настройка доступа по telnet чере Web-интерфейс](/content/images/2018/12/Screenshot-2018-12-04-22.25.16.png)

``` shell
telnet 192.168.1.1
```

Login: `root`, password: `Zte521`.

Для конфигураирования модема использвется команда `sendcmd`. Команда позволяет управлять тремя прилоджениями: `1`, `2` и `3` :).
Например, `sendcmd 1 -p` покажет список разделов (некие `PCB`). Все настройки хранятся в `DB`. Список таблиц можно получить командой `sendcmd 1 DB all`.
Данные всех таблиц можно просматривать и изменять.
Например, `sendcmd 1 DB p UserInfo` выведет все записи пользователей с паролями:

``` xml
<Tbl name="UserInfo" RowCount="4">
        <Row No="0">
                <DM name="ViewName" val="IGD.UserIF.UserInfo1"/>
                <DM name="Type" val="1"/>
                <DM name="Enable" val="1"/>
                <DM name="Username" val="admin"/>
                <DM name="Password" val="admin"/>
                <DM name="Right" val="1"/>
        </Row>
        <Row No="1">
                <DM name="ViewName" val="IGD.UserIF.UserInfo2"/>
                <DM name="Type" val="1"/>
                <DM name="Enable" val="1"/>
                <DM name="Username" val="user"/>
                <DM name="Password" val="user"/>
                <DM name="Right" val="2"/>
        </Row>
        <Row No="2">
                <DM name="ViewName" val=""/>
                <DM name="Type" val="0"/>
                <DM name="Enable" val="0"/>
                <DM name="Username" val=""/>
                <DM name="Password" val=""/>
                <DM name="Right" val="0"/>
        </Row>
        <Row No="3">
                <DM name="ViewName" val=""/>
                <DM name="Type" val="0"/>
                <DM name="Enable" val="0"/>
                <DM name="Username" val=""/>
                <DM name="Password" val=""/>
                <DM name="Right" val="0"/>
        </Row>
</Tbl>
```

Командой `sendcmd 1 DB p FWLevel` просмотреть настройки фаревола:

``` xml
<Tbl name="FWLevel" RowCount="1">
        <Row No="0">
                <DM name="Level" val="1"/>
                <DM name="AntiAttack" val="1"/>
        </Row>
</Tbl>
```

Командой `sendcmd 1 DB set FWLevel 0 AntiAttack 0` можно отключить `Anti-Hacking Protection`.
