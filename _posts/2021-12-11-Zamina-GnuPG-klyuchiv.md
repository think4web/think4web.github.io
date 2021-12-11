---
published: true
layout: post
title: Заміна GnuPG ключів
tags: gpg
discription: Заміна старих ключів GnuPG новими і підписання ключів.
---

## Відкликання ключів

Це складна і болюча частина процесу, оскільки ви фактично вилучаєте з експлуатації свій попередній ідентифікатор PGP/GPG, втрачаючи створену мережу довіри навколо нього.

Щоб повідомити світу, що ваша нова пара ключів насправді належить вам без необхідності зустрічатися особисто, стандартна процедура полягає в тому, щоб підписати новий ключ зі старим і навпаки, встановивши між ними жорсткий довірчий зв’язок. (ця дія не могла бути виконана без доступу до старого секретного ключа, тому, наскільки хтось може судити, цей двосторонній підпис міг зробити тільки ви). 

### Підписання нових і старих відкритих ключів

Перш за все, давайте перевіримо, що обидва (старий і новий) секретні ключі доступні в нашій зв'язці:

```bash
$ gpg --list-secret-keys
```

Тепер, щоб встановити довіру між новим і старим ключами, ми повинні підписати один одного. Спочатку підпишемо новий ключ старим:

```bash
$ gpg --default-key %ідентифікатор_старого_ключа% --sign-key %ідентифікатор_нового_ключа%
```

А потім навпаки: 

```bash
$ gpg --default-key %ідентифікатор_нового_ключа% --sign-key %ідентифікатор_сторого_ключа%

```

Давайте перевіримо, що новий ключ був належним чином підписаний:

```bash
$ gpg --list-sigs %ідентифікатор_нового_ключа%
```

Рядки, що містять:

sig %ідентифікатор_старого_ключа% 2019-03-27 Daniel Pecos Martinez

відображають підпис на UID. 

## Відкликаємо стару пару ключів

Тепер ми повинні ефективно вивести з експлуатації стару пару ключів. Для цього ми повинні створити для нього сертифікат відкликання:
```bash
$ gpg -a --gen-revoke BF3B5AFCD4480E60 > BF3B5AFCD4480E60.rev

sec  dsa1024/BF3B5AFCD4480E60 2002-11-11 Daniel Pecos Martinez

Create a revocation certificate for this key? (y/N) y
Please select the reason for the revocation:
  0 = No reason specified
  1 = Key has been compromised
  2 = Key is superseded
  3 = Key is no longer used
  Q = Cancel
(Probably you want to select 1 here)
Your decision? 2
Enter an optional description; end it with an empty line:
> Superseeded by E881015C8A55678B
>
Reason for revocation: Key is superseded
Superseeded by E881015C8A55678B
Is this okay? (y/N) y
Revocation certificate created.

Please move it to a medium which you can hide away; if Mallory gets
access to this certificate he can use it to make your key unusable.
It is smart to print this certificate and store it away, just in case
your media become unreadable.  But have some caution:  The print system of
your machine might store the data and make it available to others!
```

А потім імпортуйте його до нашої зв'язки, щоб фактично відкликати ключ:
```bash
$ gpg --import BF3B5AFCD4480E60.rev
gpg: key BF3B5AFCD4480E60: "Daniel Pecos Martinez" revocation certificate imported
gpg: Total number processed: 1
gpg:    new key revocations: 1
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u

$ gpg --list-secret-keys
/Users/dpecos/.gnupg/pubring.kbx
--------------------------------
sec   dsa1024 2002-11-11 [SC] [revoked: 2019-03-27]
      F6D13162F2BEF65838F6C8E6BF3B5AFCD4480E60
uid           [ revoked] Daniel Pecos Martinez
uid           [ revoked] Daniel Pecos Martinez <dpecos@gmail.com>
uid           [ revoked] Daniel Pecos Martinez <me@danielpecos.com>

sec   rsa4096 2019-03-27 [SC]
      31EFB482E969EB74399DBBC5E881015C8A55678B
uid           [ultimate] Daniel Pecos Martinez <me@danielpecos.com>
uid           [ultimate] Daniel Pecos Martinez <dani@dplabs.io>
ssb   rsa4096 2019-03-27 [E] [expires: 2024-03-25]
ssb   rsa4096 2019-03-27 [S] [expires: 2024-03-25]
```

І ми закінчили! Наступний крок – опублікувати наші зміни у світі.

Адаптація і доповнення статтей: ["How to rotate your OpenPGP / GnuPG keys"](https://danielpecos.com/2019/03/30/how-to-rotate-your-openpgp-gnupg-keys/)
