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

## Додавання налаштувань вручну

Додати у файл ```.gitconfig``` опцію ```signingkey```.

Для глобального налаштування:

```bash
[user]
	name = %User Name%
	email = %test@test.com%
	signingkey = %KeyID%
[commit]
	gpgsign = true
[core]
	autocrlf = input
```

Для локального налаштування використовується перемінна ```signingkey```, що знаходиться у файлі ```.git/config``` репозиторію.

```bash
[user]
	name = %User Name%
	email = %test@test.com%
    signingkey = %KeyID%
```

Ввімкнути обов'язковий підпис для усіх комітів:

```bash
git config --global commit.gpgsign true
```

Якщо цього не зробити то необхідно використовувати опцію "-S" у комітах.

Вказуємо де знаходиться gpg, дізнатись це можна командою ```which gpg```:

```bash
git config --global gpg.program /usr/local/bin/gpg
```

У файл конфігурації ```~/.gnupg/gpg.conf``` необхідно додати опцію ```no-tty```. Після чого треба зайти у налаштування профілю на GitHub і додати свій публічний GPG-ключ.

Для підписання комітів необхідно вводити:

```bash
git commit -S -m '%текст_коміту%'
```
Перевірка підпису:

```bash
git log --show-signature -1
```

При створенні нового коміту він буде підписаний за допомогою GPG-ключа і біля комітів з'явиться позначка **Verified**.
