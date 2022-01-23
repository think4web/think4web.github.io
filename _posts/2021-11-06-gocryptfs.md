---
published: true
layout: post
title: gocryptfs - шифрування фійлів для хмарних сервісів 
author: think4web
discription:
tags: softwate security gocryptfs
---

[gocryptfs](https://github.com/rfjakob/gocryptfs) - програма для шифрування файлів для хмарних сервісів. Заміна [CryFS](/CryFS/).

## Встановлення gocryptfs

```libpam-mount``` - дозволяє автоматично монтувати розділ при вході у систему. Також можна для зручності поставити GUI ```sirikali``` для роботи з шифрованих папок.

```bash
sudo apt install gocryptfs libpam-mount
``` 

## Налаштування gocryptfs

```gocryptfs.conf``` - конфігураційний файл.

Створюємо папку для закодованих файлів та запускаємо gocryptfs. Програма попросить ввести пароль. Для автомонтування вводимо пароль який вводили при вході в систему. Потім створюємо папку для не шифрованих файлів і монтуємо систему:

```bash
mkdir -pv ~/%шифровані_файли%
mkdir -pv ~/%не_шифровані_файли%
gocryptfs -init ~/%шифровані_файли%
gocryptfs ~/%шифровані_файли% ~/%не_шифровані_файли%
```

При монтуванні програма запитає майстер пароль і створить майстер-ключ для відновлення даних. Майстер-ключ буде виводитись у консолі при кожному монтуванні, щоб цього не відбувалося треба запускати програму з опцією **```-q```**. 

Відмотувати розділ можна командою:

```bash
fusermount -u ~/%не_шифровані_файли%
```

## Налаштування автоматичного монтування файлової системи gocryptfs

Необхідно додати у файл ```sudo gedit /etc/security/pam_mount.conf.xml``` рядок:

```bash
<volume user="%username%" fstype="fuse" options="nodev,nosuid,quiet" path="gocryptfs#/home/%(USER)/Dropbox/Encrypted" mountpoint="/home/%(USER)/Unencrypted" /&rt;
```

