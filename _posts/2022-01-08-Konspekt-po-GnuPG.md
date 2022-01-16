---
published: true
layout: post
title: Конспект по GnuPG 
author: think4web
discription:
tags: gpg
---

## Встановлення GnuPG

Для початку роботи встановіть пакет ```gnupg``` та введіть у термінал ```gpg``` щоб програма створила необхідні каталоги та файли.

## Налаштування GnuPG

Перед генерацією ключів необхідно налаштувати GnuPG. Запропоновані налаштування містять коментарі з офіційної документації GnuPG. Файл `gpg.conf` з налаштуваннями знаходиться за шляхом `~/.gnupg/gpg.conf`. Щоб перевірити де лежить конфігурація у вашій системі виконайте `gpg -homedir`. Але ми будемо виконувати усі операції на окремому носієві - USB флешці. Саме на флешці у надійному місці краще зберігати головний ключі, який не використовується у повсякденній роботі, може лише "сертифікувати" і потрібний для генерації або редагування підключів. Тому створімо на флешці папку `.gnupg` з файлом `gpg.conf` та запишемо у нього наступні налаштування.

```bash
################################################################################
# GnuPG Options
# (OpenPGP-Configuration-Options)
# Assume that command line arguments are given as UTF8 strings.
utf8-strings
# (OpenPGP-Protocol-Options)
# Set the list of personal digest/cipher/compression preferences. This allows 
# the user to safely override the algorithm chosen by the recipient key 
# preferences, as GPG will only select an algorithm that is usable by all 
# recipients.
personal-digest-preferences SHA512 SHA384 SHA256 SHA224
personal-cipher-preferences AES256 AES192 AES CAST5 CAMELLIA192 BLOWFISH TWOFISH CAMELLIA128 3DES
personal-compress-preferences ZLIB BZIP2 ZIP
# Set the list of default preferences to string. This preference list is used 
# for new keys and becomes the default for "setpref" in the edit menu. 
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed
# (OpenPGP-Esoteric-Options)
# Use name as the message digest algorithm used when signing a key. Running the 
# program with the command --version yields a list of supported algorithms. Be 
# aware that if you choose an algorithm that GnuPG supports but other OpenPGP 
# implementations do not, then some users will not be able to use the key 
# signatures you make, or quite possibly your entire key.
# 
# SHA-1 is the only algorithm specified for OpenPGP V4. By changing the 
# cert-digest-algo, the OpenPGP V4 specification is not met but with even 
# GnuPG 1.4.10 (release 2009) supporting SHA-2 algorithm, this should be safe.
# Source: https://tools.ietf.org/html/rfc4880#section-12.2
cert-digest-algo SHA512
digest-algo SHA256
# Selects how passphrases for symmetric encryption are mangled. 3 (the default) 
# iterates the whole process a number of times (see --s2k-count).
s2k-mode 3
# (OpenPGP-Protocol-Options)
# Use name as the cipher algorithm for symmetric encryption with a passphrase 
# if --personal-cipher-preferences and --cipher-algo are not given. The 
# default is AES-128. 
s2k-cipher-algo AES256
# (OpenPGP-Protocol-Options)
# Use name as the digest algorithm used to mangle the passphrases for symmetric 
# encryption. The default is SHA-1. 
s2k-digest-algo SHA512
# (OpenPGP-Protocol-Options)
# Specify how many times the passphrases mangling for symmetric encryption is 
# repeated. This value may range between 1024 and 65011712 inclusive. The 
# default is inquired from gpg-agent. Note that not all values in the 
# 1024-65011712 range are legal and if an illegal value is selected, GnuPG will 
# round up to the nearest legal value. This option is only meaningful if 
# --s2k-mode is set to the default of 3. 
s2k-count 1015808
################################################################################
# GnuPG View Options
# Select how to display key IDs. "long" is the more accurate (but less 
# convenient) 16-character key ID. Add an "0x" to include an "0x" at the 
# beginning of the key ID.
keyid-format 0xlong
# List all keys with their fingerprints. This is the same output as --list-keys 
# but with the additional output of a line with the fingerprint. If this 
# command is given twice, the fingerprints of all secondary keys are listed too.
with-fingerprint
with-fingerprint
```
Коментар пояснює чому були обрані параметри `cert-digest-algo` і `digest-algo` навіть попри те що вони порушують специфікацію OpenPGP.

Останні параметри з прикладу вище впливає на те, як ключі відображаються у виводі GnuPG. Рекомендується використовувати формат `long keyid` для ідентифікації ключів. Keyid – це коротке зображення відбитку (fingerprint). Довгий формат бере останні 16 (замість 8 у короткому форматі) символів з відбитку. Так ви будете дійсно впевненими що маєте на увазі правильний ключ, fingerprint – це інформація для ідентифікації ключа.

## Генерація майстер-ключа

Для прикладу додамо електронну пошту під час створення головного ключа, але краще не робити так і додати усю необхідну інформацію пізніше при редагуванні, а зразу обмежитись тільки ім'ям. Якщо для побудови мережі довіри і завірення ключів вам небідно буде підтверджувати свою особу та пред'являти документи, використовуйте під час створення ключа ім'я на яке у вас є документи.  

Якщо при генерації станеться помилка:
```bash
gpg: agent_genkey failed: No such file or directory
Key generation failed: No such file or directory
```

Просто вбийте ``` gpg-agent``` командою ```gpgconf --kill gpg-agent```.

Нагадаю, що ключі генеруються без підключення до мережі інтернет.

```bash
$ gpg --homedir /%флешка%/.gnupg --expert --full-generate-key
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
Your selection? 8
Possible actions for a RSA key: Sign Certify Encrypt Authenticate 
Current allowed actions: Sign Certify Encrypt 
   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished
Your selection? S
Possible actions for a RSA key: Sign Certify Encrypt Authenticate 
Current allowed actions: Certify Encrypt 
   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished
Your selection? E
Possible actions for a RSA key: Sign Certify Encrypt Authenticate 
Current allowed actions: Certify 
   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished
Your selection? Q
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) Y
You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"
Real name: Alice
Email address:
Comment: 
You selected this USER-ID:
    "Alice"
Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
You need a Passphrase to protect your secret key.
Enter passphrase: YourPassword
Repeat passphrase: YourPassword
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
.+++++
...................+++++
gpg: key 0xD93D03C13478D580 marked as ultimately trusted
public and secret key created and signed.
gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: next trustdb check due at 2018-11-30
pub   4096R/0xD93D03C13478D580 2016-11-30 [expires: 2018-11-30]
      Key fingerprint = F8C8 1342 2A7F 7A3A 9027  E158 D93D 03C1 3478 D580
uid                            Alice
```

Відповідно до параметрів перегляду у файлі конфігурації `gpg.conf`, у виводі буде показано довгий формат ключа ключа, а також відбиток кожного ключа чи підключа.
Для перевірки згенерованих ключів у зв'язці виконайте таку команду. 

```bash
$ gpg --homedir /%флешка%/.gnupg -K
/%флешка%/.gnupg/secring.gpg
------------------------
sec   4096R/0xD93D03C13478D580 2016-11-30 [expires: 2018-11-30]
      Key fingerprint = F8C8 1342 2A7F 7A3A 9027  E158 D93D 03C1 3478 D580
uid                            Alice
```

## Налаштування параметрів ключа

Щоб переконатися, що використовуються тільки надійні алгоритми, встановіть параметри для ключа за допомогою команди «setpref». 

```bash
$ gpg --homedir /%флешка%/.gnupg --expert --edit-key 0xD93D03C13478D580
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Secret key is available.
pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C   
                               trust: ultimate      validity: ultimate
[ultimate] (1). Alice
gpg> setpref SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed
Set preference list to:
     Cipher: AES256, AES192, AES, CAST5, 3DES
     Digest: SHA512, SHA384, SHA256, SHA224, SHA1
     Compression: ZLIB, BZIP2, ZIP, Uncompressed
     Features: MDC, Keyserver no-modify
Really update the preferences? (y/N) Y
You need a passphrase to unlock the secret key for
user: Alice
4096-bit RSA key, ID 0xD93D03C13478D580, created 2016-11-30
Enter passphrase: YourPassword
pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C   
                               trust: ultimate      validity: ultimate
[ultimate] (1). Alice
gpg> save
```

## Генерація підключів

Підключі використовуються щоб не наражати на небезпеку основний ключ. Навіть у випадку компрометації підключа, його можна просто відкликати без необхідності відкликати основний ключ. Кількість підключів має відповідати кількості пристроїв на яких ви плануєте ними користуватися і їх призначенню (шифрування, підпис, автентифікації) так щоб для кожного пристрою були свій ключі.

Перш ніж почати генерувати різні підключі, перевірте максимальний розмір, який може зберігати смарт-карта. Yubikey NEO може зберігати ключі до 2048 біт, а Yubikey 4 може зберігати ключі до 4096 біт. Смарт-карти зазвичай підтримують різні розміри: 2048, 3072 або 4096 біт.

Щоб додати підключ, головний ключ потрібно відкрити для редагування. Наступна команда відкриє вказаний ключ (у наведеному нижче прикладі через ідентифікатор ключа) для редагування. Щоб мати можливість створити всі різні типи ключів, використовується параметр `–expert`.

```bash
$ gpg --homedir /%флешка%/.gnupg --expert --edit-key 0xD93D03C13478D580
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Secret key is available.
pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C   
                               trust: ultimate      validity: ultimate
[ultimate] (1). Alice
gpg> save
```

### Підключ для підпису

У відкритий для редагування ключ можна додати підключ. Щоб почати керований процес створення підключа, введіть команду `addkey`.
Після введення парольної фрази необхідно ввести тип підключа. Для ключа яким будемо підписувати використовується `(4) RSA (лише підписи)`. Розмір ключа повинен відповідати розміру доступному на смарт-картці або Yubikey, якщо розмір не має значення, краще використовувати 3072 або 4096 біт.

У той час як GnuPG першої версії запитуватиме парольну фразу на початку процедури додавання ключа, друга версія запитуватиме в кінці процесу створення індивідуальну парольну фразу для нового підключа, а також парольну фразу головного ключа.

```bash
gpg> addkey
Key is protected.
You need a passphrase to unlock the secret key for
user: "Alice"
4096-bit RSA key, ID 0xD93D03C13478D580, created 2016-11-30
Enter passphrase: YourPassword
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
Your selection? 4
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Fri 30 Nov 2018 10:44:21 PM CET
Is this correct? (y/N) Y
Really create? (y/N) Y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
............+++++
+++++
pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C   
                               trust: ultimate      validity: ultimate
sub  4096R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S   
[ultimate] (1). Alice
gpg> 
```

### Підключ для шифрування
Тепер можна створити підключ для шифрування таким же чином як раніше ми створювали підключ для підпису. Просто вибираємо тип ключа `RSA (лише шифрування)`. 
```bash
gpg> addkey
Key is protected.
You need a passphrase to unlock the secret key for
user: "Alice <alice@example.com>"
4096-bit RSA key, ID 0xD93D03C13478D580, created 2016-11-30
Enter passphrase: YourPassword
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
Your selection? 6
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Fri 30 Nov 2018 10:44:23 PM CET
Is this correct? (y/N) Y
Really create? (y/N) Y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
.......+++++
....................+++++
pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C   
                               trust: ultimate      validity: ultimate
sub  4096R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S   
sub  4096R/0x76737ABEB92745D7  created: 2016-11-30  expires: 2018-11-30  usage: E   
[ultimate] (1). Alice
gpg> 
```

### Підключ автентифікації

Якщо для автентифікації ви плануєте використовувати ключ GnuPG, створюємо додатковий підключ автентифікації. Таким підключем  можна скористатися для автентифікації, наприклад, при підключенні через SSH.

Тип (8) дозволяє налаштувати призначення ключа вручну. Список вище показує варіанти призначення підключ. За замовчуванням ключ здатний «Підписати» та «Шифрувати».
Вимкніть призначення за замовчуванням, а потім вводьте відповідну літеру та ENTER, вибираючи одне призначення за іншим. Використовуйте перемикач «A», щоб увімкнути можливість автентифікації, і перейдіть до «Q» щоб завершити генерацію. 
Щоб створити підключ автентифікації, потрібно вибрати тип «(8) RSA (встановити власні можливості)».

```bash
gpg> addkey
Key is protected.
You need a passphrase to unlock the secret key for
user: "Alice"
4096-bit RSA key, ID 0xD93D03C13478D580, created 2016-11-30
Enter passphrase: YourPassword
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
Your selection? 8
Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: Sign Encrypt 
   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished
Your selection?
Your selection? S
Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: Encrypt 
   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished
Your selection? E
Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: 
   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished
Your selection? A
Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: Authenticate 
   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished
Your selection? Q
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Fri 30 Nov 2018 10:44:26 PM CET
Is this correct? (y/N) Y
Really create? (y/N) Y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
..+++++
.....+++++
pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C   
                               trust: ultimate      validity: ultimate
sub  4096R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S   
sub  4096R/0x76737ABEB92745D7  created: 2016-11-30  expires: 2018-11-30  usage: E   
sub  4096R/0xE379FB0D81B6925D  created: 2016-11-30  expires: 2018-11-30  usage: A   
[ultimate] (1). Alice
gpg> save
```

Як ми бачимо, головний ключ та підключі мають 4096 біт, але можна зробити 3 підключа різного розміру, щбо пізніше ми встановити їх на смарт-картку або Yubikey.

## Додайте додаткову інформацію

До ключа GnuPG можна додати більше інформації щоб його легше було упізнати. На кшалт додавання додаткової особи до ключа. Для цього ключ потрібно знову відкрити в режимі редагування. Тоді команда «adduid» використовується для додавання додаткової інформації. Саме тут ми додамо адресу пошти, якщо не зробили цього при генерації головного ключа. 

```bash 
$ gpg --homedir /%флешка%/.gnupg --expert --edit-key 0xD93D03C13478D580
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Secret key is available.
pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C   
                               trust: ultimate      validity: ultimate
sub  4096R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S   
sub  4096R/0x76737ABEB92745D7  created: 2016-11-30  expires: 2018-11-30  usage: E   
sub  4096R/0xE379FB0D81B6925D  created: 2016-11-30  expires: 2018-11-30  usage: A   
[ultimate] (1). Alice <alice@example.com>
gpg> adduid
Real name:
Email address: alice@example.org
Comment: 
You selected this USER-ID:
    "Alice"
Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
You need a passphrase to unlock the secret key for
user: "Alice <alice@example.com>"
4096-bit RSA key, ID 0xD93D03C13478D580, created 2016-11-30
Enter passphrase: YourPassword
pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C   
                               trust: ultimate      validity: ultimate
sub  4096R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S   
sub  4096R/0x76737ABEB92745D7  created: 2016-11-30  expires: 2018-11-30  usage: E   
sub  4096R/0xE379FB0D81B6925D  created: 2016-11-30  expires: 2018-11-30  usage: A   
[ultimate] (1)  Alice
[ unknown] (2). alice@example.org
gpg> uid 1
pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C
                               trust: ultimate      validity: ultimate
sub  4096R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S
sub  4096R/0x76737ABEB92745D7  created: 2016-11-30  expires: 2018-11-30  usage: E
sub  4096R/0xE379FB0D81B6925D  created: 2016-11-30  expires: 2018-11-30  usage: A
[ultimate] (2)* alice@example.com
[ultimate] (1).
gpg> primary
You need a passphrase to unlock the secret key for
user: "Alice"
4096-bit RSA key, ID 0xD93D03C13478D580, created 2016-11-30
Enter passphrase: YourPassword
pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C
                               trust: ultimate      validity: ultimate
sub  4096R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S
sub  4096R/0x76737ABEB92745D7  created: 2016-11-30  expires: 2018-11-30  usage: E
sub  4096R/0xE379FB0D81B6925D  created: 2016-11-30  expires: 2018-11-30  usage: A
[ultimate] (2)* alice@example.com
[ultimate] (1)  Alice
gpg> save
```

Останній вихід після додавання ідентифікатора показує ключ із довірою «unknown», який зміниться на необмежений після збереження нового посвідчення за допомогою команди «save».

Позначення "." показує головний ідентифікатор для цього ключа. Як показано вище, після додавання другої особи, доданий ідентифікатор вибирається як основний. Але ми визначили основний ідентифікатор вручну шляхом вибору потрібного посвідчення за допомогою команди «uid», а потім команди «primary», щоб встановити цю особистість як первинну. Якщо ми додавали тільки пошту, головним записом буде запис з ім'ям.

## Експортуємо резервну копію публічного та секретного ключів

Щоб створити резервну копію ключів, експортуйте їх у файл. Експорт ключів виконується в два етапи, приватні ключі та секретні ключі експортуються окремо. 

```bash
$ gpg --homedir /%флешка%/.gnupg --export-secret-keys --armor --output ~/.gnupg/secret-keys.gpg 0xD93D03C13478D580
$ gpg --homedir /%флешка%/.gnupg --export --armor --output ~/.gnupg/public-keys.gpg 0xD93D03C13478D580
```

За допомогою першої команди всі секретні ключі (головні + підключі) експортуються в один файл. Друга команда експортує всі публічні ключі (головні + підключі) в інший файл. Ці файли можна використовувати для резервного копіювання створених ключів зберігши їх на окремий носій.

Щоб експортувати лише один конкретний підключ, ідентифікатор підключа можна вказати за допомогою «!» Знак оклику в кінці ID ключа вказує gpg експортувати лише цей конкретний підключ. Це доцільно зробити для підключів які будуть використовуватись на окремих пристроях. 

```bash
$ gpg --homedir /%флешка%/.gnupg --export-secret-subkeys --armor --output ~/.gnupg/secret-subkeys/0x1ED73636975EC6DE.asc 0x1ED73636975EC6DE!
$ gpg --homedir /%флешка%/.gnupg --export --armor --output ~/.gnupg/public-keys/0xD93D03C13478D580.asc 0xD93D03C13478D580!
```

Наведена вище команда експортує лише секретний ключ підключа для підпису. Довідкова сторінка gpg описує знак оклику в ідентифікаторі ключа так.

"Під час використання gpg знак оклику (!) може бути доданий, щоб примусово використовувати вказаний первинний або вторинний ключ, а не намагатися обчислити, який первинний або вторинний ключ використовувати." 

## Зв'язка ключів для щоденного використання

Для щоденного використання необхідно «вилучити» головний секретний ключ зі зв'язки щоб ризикувати ним. Таку зв'язку можна підготувати кількома шляхами.

### Варіант 1. Видалення секретного головного ключа

Експортуємо лише секретні підключі, видаляємо всі секретні частини ключів цього ключа з комплекту ключів (що включає не тільки головний ключ, але й підключі), а потім повторно імпортуючи лише секретні підключів залишаючи зв'язку без секретної частини головного ключа. 

```bash
$ gpg --homedir /%флешка%/.gnupg --export-secret-subkeys --armor --output ~/.gnupg/secret-subkeys.gpg 0xD93D03C13478D580
```

Ця команда експортує всі секретні підключі заданого ідентифікатора ключа та збереже його у вказаному файлі. 
GnuPG другої версії просить видаляти кожен секретний ключ/підключ. На цьому етапі можна прийняти операцію видалення головного ключа, але видалити підключі можна заборонити. Це призведе до повідомлення про помилку для операції видалення, про те що підключи не були видалені. Так можна видалити секретний головний ключ, але зберегти підключі, а отже, не буде потреби повторного імпорту секретних підключів.

```bash
$ gpg --homedir ~/.gnupg --delete-secret-keys 0xD93D03C13478D580
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
sec  4096R/0xD93D03C13478D580 2016-11-30 Alice <alice@example.org>
Delete this key from the keyring? (y/N) Y
This is a secret key! - really delete? (y/N) Y
```

Після виконання наведеної вище команди всі секретні головні ключі видаляються зі зв'язки. За допомогою наступної команди експортовані секретні підключі повторно імпортуються назад до зв'язки.

```bash
$ gpg --homedir /%флешка%/.gnupg --import ~/.gnupg/secret-subkeys.gpg
gpg: key 0xD93D03C13478D580: secret key imported
gpg: key 0xD93D03C13478D580: "Alice <alice@example.org>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
```

### Варіант 2. Створення з резервної копії

Якщо зв'язка ключів яку ви використовували для створення, не є тією зв'язкою яку ви збираєтеся використовувати щоденно, файли резервної копії, створені раніше, можна використовувати для створення щоденного зв'язки. Це найкращий метод, якщо ви створили ключ не на комп'ютері яким ви користуєтеся для повсякденних завдань, або створили ключ на USB-накопичувачеві чи тому подібному.

```bash
$ gpg --homedir /%флешка%/.gnupg --export-secret-subkeys --armor --output ~/.gnupg/secret-subkeys.gpg 0xD93D03C13478D580
```

Як і в першому варіанті, експорт лише з підключами має бути здійснений за допомогою команди вище. На додаток до наступних команд, я пропоную також скопіювати **gpg.conf**, який використовується в наборі ключів, щоб створити ключ для щоденного використання.

```bash
$ gpg --homedir ~/.gnupg --import %флешка%/public-keys.gpg
gpg: key 0xD93D03C13478D580: public key "Alice <alice@example.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
```

Наведену вище команду можна використовувати для імпорту публічного ключа до зв'язки, яку ви використовуєте щоденно. За допомогою наведеної нижче команди секретні підключі без головного секретного ключа можна імпортувати до щоденного використання ключів. «–homedir ~/.gnupg» можна опустити, якщо зв'язка знаходиться за замовчуванням, як у цьому прикладі. 

```bash
$ gpg --homedir ~/.gnupg --import %флешка%/secret-subkeys.gpg
gpg: key 0xD93D03C13478D580: secret key imported
gpg: key 0xD93D03C13478D580: "Alice <alice@example.org>" 1 new signature
gpg: Total number processed: 1
gpg:         new signatures: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
```

## Перевірте зв'язку для щоденного використання

Щоб переконатися, що тільки секретні ключі підключів було імпортовано назад у кільце ключів, виконайте таку команду. Ця команда покаже список усіх секретних ключів. Головний ключ позначений символом хеша «#», що вказує на те, що секретний ключ відсутній – як і очікувалося.

```bash
$ gpg --homedir ~/.gnupg -K
~/.gnupg/secring.gpg
------------------------
sec#  4096R/0xD93D03C13478D580 2016-11-30 [expires: 2018-11-30]
      Key fingerprint = F8C8 1342 2A7F 7A3A 9027  E158 D93D 03C1 3478 D580
uid                            Alice
uid                            alice@example.org
ssb   4096R/0x1ED73636975EC6DE 2016-11-30
      Key fingerprint = 292D 3E78 6B2E DBEA 1D10  02C8 1ED7 3636 975E C6DE
ssb   4096R/0x76737ABEB92745D7 2016-11-30
      Key fingerprint = 0C33 42E5 670A B099 8ED7  3E87 7673 7ABE B927 45D7
ssb   4096R/0xE379FB0D81B6925D 2016-11-30
      Key fingerprint = 7357 2158 947D BAFF A89F  4911 E379 FB0D 81B6 925D
```

Нарешті, зберігайте резервну копію секретних ключів, зокрема секретного головного ключа, і видаліть будь-який інший тимчасовий файл експорту секретних підключів, який більше не потрібен. Розгляньте можливість використання утиліти [shred](https://linux.die.net/man/1/shred), щоб безпечно видалити ці файли. 

## Клієнти

[OpenKeychain](https://f-droid.org/packages/org.sufficientlysecure.keychain/)

Переклад і доповнення статтей: 
- ["Create GnuPG key with sub-keys to sign, encrypt, authenticate"](https://blog.tinned-software.net/create-gnupg-key-with-sub-keys-to-sign-encrypt-authenticate/)
