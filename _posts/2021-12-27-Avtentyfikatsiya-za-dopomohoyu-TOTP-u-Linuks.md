---
published: true
layout: post
title: Автентифікація за допомогою ТОТP у Linux
author: think4web
discription: Налаштування одноразових кодів TOTP для входу у систему Linux.
tags: software 
---

Встановлюємо PAM-модуль [google-authenticator-libpam](https://github.com/google/google-authenticator-libpam):

```bash
git clone https://github.com/google/google-authenticator-libpam.git
cd google-authenticator-libpam
./bootstrap.sh
./configure
make
sudo make install
```

Запускаємо програму ```google-authenticator```. Має згенеруватися QR-код або з'явитися посилання на нього, який необхідно відсканувати за допомогою програми для TOTP-автентифікації. На усі питання програми, крім питання про збільшення часового розходження між клієнтом і сервером, відповідати негативно.

Додаємо у файл ```/etc/pam.d/common-auth``` наступний рядок:

```bash
auth required pam_google_authenticator.so no_increment_hotp
```

Цей спосіб можна комбінувати з введенням паролю та іншими способами двофакторної автентифікації.
