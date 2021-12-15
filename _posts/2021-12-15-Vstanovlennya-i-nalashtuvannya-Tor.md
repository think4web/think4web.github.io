---
published: true
layout: post
title: Встановлення і налаштування Tor
author: think4web
discription: Встановлення і налаштування браузера Tor у GNU/Linux.
tags: linux tor privacy security
---

## Встановлюємо Tor

Для встановлення ```torbrowser``` необхідно підключити нестабільну гілку репозиторію і налаштувати її так щоб з неї встановлювався тільки ```torbrowser-launcher```. Вводимо ```sudo nano /etc/apt/preferences``` і редактуємо налаштування наступним чином:

```bash
Package: *
Pin: release n=sid
Pin-Priority: -1

Package: torbrowser-launcher
Pin: release n=sid
Pin-Priority: 700
```

Тепер додаємо у ```sources.list``` такий рядок:

```bash
deb tor+https://deb.debian.org/debian sid main contrib
``` 

Оновлюємося і встановлюємо лаунчер Tor: ```sudo apt install torbrowser-launcher```.

## Налаштовуємо Tor

Запускаємо лаунчер і чекаємо встановлення браузеру. Після встановлення натискаємо на емблему щіту і вибираємо максимальну безпеку "Safest". Це блокує запуск скриптів, але деякі сайти не будуть працювати правильно. Альтернативним варіантом буде доповнення [NoScript](https://noscript.net/).

Разом з TorBrowser встановлюються apparmor-профілі які блокують доступ з браузеру до домашньої директорії користувача. Для перевірки введіть в адрисний рядок ```/home```, має з'явитися помилка.
