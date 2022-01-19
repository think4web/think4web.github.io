---
published: true
layout: post
title: Налаштування GnuPG для GitHub
author: think4web
discription: Як налаштувати підпис комітів у GitHub своїм GPG-ключем. 
tags: gpg github
---

## Налаштування автоматичного підписання комітів

Для локальних налаштувань використати опцію ```--local```, для глобальних - опцію ```--global```

Глобально додати ключ для підписання у репозиторію:

```bash
git config --global user.signingkey %KeyID%
```

Глобально ввімкнути обов'язковий підпис для усіх комітів:

```bash
git config --global commit.gpgsign true
```

Глобально вказати де знаходиться GPG, дізнатись це можна командою ```which gpg```:

```bash
git config --global gpg.program /usr/local/bin/gpg
```

## Файли конфігурації

Глобальний файл конфігурації git знаходиться за адресою ```~/.gitconfig```.

Локальний файл конфігурації git знаходиться за адресою ```git/comfig``` у самому репозиторії.

У файл конфігурації треба додати перемінну **signingkey**:

```bash
[user]
  name = %User Name%
  email = %user@email.com%
 signingkey = %KeyID%
[commit]
	gpgsign = true
[core]
	autocrlf = input
```

## Налаштування GitHub

У налаштування профілю на GitHub додати публічний GPG-ключ для підпису. У ідентифікаторах ключа (**uid**) має бути вказана та сама пошта що вказана в GitHub у якості публічної пошти.

## Підписання коментарів вручну

Для примусового підписання комітів необхідно вводити ключ **-S**:

```bash
git commit -S -m '%текст_коміту%'
```

Перевірка підпису:

```bash
git log --show-signature -1
```

При створенні нового коміту він буде підписаний за допомогою GPG-ключа і біля комітів з'явиться позначка **Verified**.
