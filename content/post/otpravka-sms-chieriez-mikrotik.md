---
title: "Отправка смс через mikrotik"
date: 2018-07-15
draft: false
description: "Mikrotik - отправка SMS (СМС)"
tags: [
	"mikrotik",
	"sms"
]
---

[Use_SSH_to_execute_commands](http://wiki.mikrotik.com/wiki/Use_SSH_to_execute_commands_(DSA_key_login))

``` shell
ssh router '/tool sms send usb1 "+375XXXXXXXXXX" message="Hello, world"'
```
