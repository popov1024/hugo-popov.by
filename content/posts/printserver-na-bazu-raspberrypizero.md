---
title: Printserver на базу RaspberryPi
date: 2019-07-09
draft: false
---

Raspberry Pi отлично подходит для принтсервера. В моем случае используется RaspberryPi Zero W, подключаемый к сети по WiFi.

![printsert-raspberrypi](/images/printsert-raspberrypi.jpg)

Для установки CUPS используем Ansible:

``` ansible
---
- name: Install cups
  apt:
    name: cups
    state: latest
  become: yes

- name: setup user pi for cups
  shell: sudo usermod -a -G lpadmin pi
  become: yes

- name: Install cups drivers
  apt:
    name:
    - printer-driver-foo2zjs-common
    - printer-driver-foo2zjs
    - xinetd
    state: latest
  become: yes

- name: setup remote connecting to any
  shell: cupsctl --remote-any
  become: yes

- name: restart_cups
  shell: /etc/init.d/cups restart
  become: yes
```

На NFC метку можно записать url  админки CUPS.
