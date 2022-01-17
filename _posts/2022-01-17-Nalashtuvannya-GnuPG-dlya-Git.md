---
published: false
layout: post
title: Налаштування GnuPG для Git
author: think4web
discription:
tags: gpg
---

## Налаштування автоматичного підписання комітів

```bash
git config --global user.signingkey %KeyID%
git config --global commit.gpgsign true
git config --global gpg.program $(which gpg2)
```

Після чого треба зайти у налаштування профілю на GitHub і додати свій GPG-ключ.

При створенні нового коміту він буде підписаний за допомогою GPG-ключа і біля комітів з'явиться позначка **Verified**.

https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits
