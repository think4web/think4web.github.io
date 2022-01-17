---
published: true
layout: post
title: Налаштування GnuPG для Git
author: think4web
discription:
tags: gpg
---

## Налаштування автоматичного підписання комітів

Додає ключ для підписання усіх репозиторіїв:

```bash
git config --global user.signingkey %KeyID%
```

Додати ключ для підписання локального репозиторію:

```bash
git config --local user.signingkey %KeyID%
```

```bash
git config --global commit.gpgsign true
git config --global gpg.program /usr/local/bin/gpg
```

У файл конфігурації ```~/.gnupg/gpg.conf``` необхідно додати опцію ```no-tty```. Після чого треба зайти у налаштування профілю на GitHub і додати свій публічний GPG-ключ.

Для підписання комітів необхідно вводити:

```bash
git commit -S -m '%текст_коміту%'
```

При створенні нового коміту він буде підписаний за допомогою GPG-ключа і біля комітів з'явиться позначка **Verified**.
