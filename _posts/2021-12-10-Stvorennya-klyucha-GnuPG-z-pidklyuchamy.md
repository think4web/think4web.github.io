---
published: true
layout: post
title: Створення ключа GnuPG з підключами
tags: gpg
discription: Створення ключа GnuPG з підключами для підпису, шифрування, автентифікації.
---

Створимо ключ GnuPG для використання його на смарт-картці, Yubikey, або просто на звичайній флешці.  Ця публікація покаже вам, як створити ключ GnuPG з підключами для підписання, шифрування та автентифікації Ключ автентифікації підходить для автентифікації через SSH.

## Налаштуйте GnuPG

Перед генерацією ключів необхідно налаштувати GnuPG. Запропоновані налаштування містять коментарі з офіційної документації GnuPG. Файл `gpg.conf` з налаштуваннями знаходиться за шляхом `~/.gnupg/gpg.conf`. Щоб перевірити де лежить конфігурація у вашій системі виконайте `gpg -homedir`. Але ми будемо виконувати усі операції на окремому носієві - USB флешці. Саме на флешці у надійному місці краще зберігати головний ключі, який не використовується у повсякденній роботі, може лише "сертифікувати" і потрібний для генерації або редагування підключів. Тому створімо на флешці папку `.gnupg` з файлом `gpg.conf` та запишемо у нього наступні налаштування.

```
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

У прикладі далі описано як створити 4096-бітний ключ головний ключ. Параметр `–homedir ./gnupg-test/` визначає каталог, який використовується для створення зв'язки. У цьому каталогу має знаходитись раніше створений файл gpg.conf. 

Генерування випадкових даних, необхідних для генерації ключа, може зайняти багато часу. Щоб прискорити процес, використовуйте rngd(8) для подачі випадкових даних у пул випадкових чисел ядра. Це можна зробити, встановивши пакет `rng-tools`.

```bash
# Debian / Ubuntu
$ sudo apt-get install rng-tools

# RedHat / CentOS
$ yum install rng-tools
```

У OpenPGP 2.x, яка називається `gpg2` у багатьох дистрибутивах, параметр для створення ключа змінився на `–full-gen-key`. Якщо з gpg2 використовується параметр `–gen-key`, багато параметрів з прикладу будуть недоступні.

Для прикладу додамо електронну пошту під час створення головного ключа, але краще не робити так і додати усю необхідну інформацію пізніше при редагуванні, а зразу обмежитись тільки ім'ям. Якщо для побудови мережі довіри і завірення ключів вам небідно буде підтверджувати свою особу та пред'являти документи, використовуйте під час створення ключа ім'я на яке у вас є документи.  

Нагадаю, що ключі генеруються без підключення до мережі інтернет.

```bash
$ gpg --homedir ./gnupg-test --expert --gen-key
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
Key is valid for? (0) 2y
Key expires at Fri 30 Nov 2018 10:44:14 PM CET
Is this correct? (y/N) Y

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

Real name: Alice
Email address: alice@example.com
Comment: 
You selected this USER-ID:
    "Alice <alice@example.com>"

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
uid                            Alice <alice@example.com>
```

Відповідно до параметрів перегляду у файлі конфігурації `gpg.conf`, у виводі буде показано довгий формат ключа ключа, а також відбиток кожного ключа чи підключа.

Для перевірки згенерованих ключів у зв'язці виконайте таку команду. 

```bash
$ gpg --homedir ./gnupg-test -K
./gnupg-test/secring.gpg
------------------------
sec   4096R/0xD93D03C13478D580 2016-11-30 [expires: 2018-11-30]
      Key fingerprint = F8C8 1342 2A7F 7A3A 9027  E158 D93D 03C1 3478 D580
uid                            Alice <alice@example.com>
```

## Налаштування параметрів ключа

Щоб переконатися, що використовуються тільки надійні алгоритми, встановіть параметри для ключа за допомогою команди «setpref». 

```bash
$ gpg --homedir ./gnupg-test --expert --edit-key 0xD93D03C13478D580
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C   
                               trust: ultimate      validity: ultimate
[ultimate] (1). Alice <alice@example.com>

gpg> setpref SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed
Set preference list to:
     Cipher: AES256, AES192, AES, CAST5, 3DES
     Digest: SHA512, SHA384, SHA256, SHA224, SHA1
     Compression: ZLIB, BZIP2, ZIP, Uncompressed
     Features: MDC, Keyserver no-modify
Really update the preferences? (y/N) Y

You need a passphrase to unlock the secret key for
user: "Alice <alice@example.com>"
4096-bit RSA key, ID 0xD93D03C13478D580, created 2016-11-30

Enter passphrase: YourPassword

pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C   
                               trust: ultimate      validity: ultimate
[ultimate] (1). Alice <alice@example.com>

gpg> save
```

## Генерація підключів

Підключі використовуються щоб не наражати на небезпеку основний ключ. Навіть у випадку компрометації підключа, його можна просто відкликати без необхідності відкликати основний ключ. Кількість підключів має відповідати кількості пристроїв на яких ви плануєте ними користуватися і їх призначенню (шифрування, підпис, автентифікації) так щоб для кожного пристрою були свій ключі.

Перш ніж почати генерувати різні підключі, перевірте максимальний розмір, який може зберігати смарт-карта. Yubikey NEO може зберігати ключі до 2048 біт, а Yubikey 4 може зберігати ключі до 4096 біт. Смарт-карти зазвичай підтримують різні розміри: 2048, 3072 або 4096 біт.

Щоб додати підключ, головний ключ потрібно відкрити для редагування. Наступна команда відкриє вказаний ключ (у наведеному нижче прикладі через ідентифікатор ключа) для редагування. Щоб мати можливість створити всі різні типи ключів, використовується параметр `–expert`.

```bash
$ gpg --homedir ./gnupg-test --expert --edit-key 0xD93D03C13478D580
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C   
                               trust: ultimate      validity: ultimate
[ultimate] (1). Alice <alice@example.com>

gpg> 
```

### Підключ для підпису

У відкритий для редагування ключ можна додати підключ. Щоб почати керований процес створення підключа, введіть команду `addkey`.

Після введення парольної фрази необхідно ввести тип підключа. Для ключа яким будемо підписувати використовується `(4) RSA (лише підписи)`. Розмір ключа повинен відповідати розміру доступному на смарт-картці або Yubikey, якщо розмір не має значення, краще використовувати 3072 або 4096 біт.

У той час як GnuPG першої версії запитуватиме парольну фразу на початку процедури додавання ключа, друга версія запитуватиме в кінці процесу створення індивідуальну парольну фразу для нового підключа, а також парольну фразу головного ключа.

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
Your selection? 4
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 3072
Requested keysize is 3072 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2y
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
sub  3072R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S   
[ultimate] (1). Alice <alice@example.com>

gpg> 
```

Наведений вище вивід вже показує додатковий підключ для підпису («використання: S»). Якщо більше не треба додавати підключі, процес можна завершити командою «save», щоб зберегти зміни до основного ключа.

```bash
gpg> save
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
What keysize do you want? (2048) 3072
Requested keysize is 3072 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2y
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
sub  3072R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S   
sub  3072R/0x76737ABEB92745D7  created: 2016-11-30  expires: 2018-11-30  usage: E   
[ultimate] (1). Alice <alice@example.com>

gpg> 
```

### Підключ автентифікації

Якщо для автентифікації ви плануєте використовувати ключ GnuPG, створюємо додатковий підключ автентифікації. Таким підключем  можна скористатися для автентифікації, наприклад, при підключенні через SSH.

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
Your selection? 8

Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: Sign Encrypt 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection?
```

Тип (8) дозволяє налаштувати призначення ключа вручну. Список вище показує варіанти призначення підключ. За замовчуванням ключ здатний «Підписати» та «Шифрувати».

Вимкніть призначення за замовчуванням, а потім вводьте відповідну літеру та ENTER, вибираючи одне призначення за іншим. Використовуйте перемикач «A», щоб увімкнути можливість автентифікації, і перейдіть до «Q» щоб завершити генерацію. 

Щоб створити підключ автентифікації, потрібно вибрати тип «(8) RSA (встановити власні можливості)».

```bash
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
What keysize do you want? (2048) 3072
Requested keysize is 3072 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2y
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
sub  3072R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S   
sub  3072R/0x76737ABEB92745D7  created: 2016-11-30  expires: 2018-11-30  usage: E   
sub  3072R/0xE379FB0D81B6925D  created: 2016-11-30  expires: 2018-11-30  usage: A   
[ultimate] (1). Alice <alice@example.com>

gpg> save
```

Як ми бачимо, головний ключ має 4096 біт, а 3 підключі мають різні розміри, бо пізніше ми встановимо їх на смарт-картку або Yubikey.

## Додайте додаткову інформацію

До ключа GnuPG можна додати більше інформації щоб його легше було упізнати. На кшалт додавання додаткової особи до ключа. Для цього ключ потрібно знову відкрити в режимі редагування. Тоді команда «adduid» використовується для додавання додаткової інформації. Саме тут ми додамо адресу пошти, якщо не зробили цього при генерації головного ключа. 

```bash 
$ gpg --homedir ./gnupg-test --expert --edit-key 0xD93D03C13478D580
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C   
                               trust: ultimate      validity: ultimate
sub  3072R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S   
sub  3072R/0x76737ABEB92745D7  created: 2016-11-30  expires: 2018-11-30  usage: E   
sub  3072R/0xE379FB0D81B6925D  created: 2016-11-30  expires: 2018-11-30  usage: A   
[ultimate] (1). Alice <alice@example.com>

gpg> adduid
Real name: Alice
Email address: alice@example.org
Comment: 
You selected this USER-ID:
    "Alice <alice@example.org>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O

You need a passphrase to unlock the secret key for
user: "Alice <alice@example.com>"
4096-bit RSA key, ID 0xD93D03C13478D580, created 2016-11-30

Enter passphrase: YourPassword

pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C   
                               trust: ultimate      validity: ultimate
sub  3072R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S   
sub  3072R/0x76737ABEB92745D7  created: 2016-11-30  expires: 2018-11-30  usage: E   
sub  3072R/0xE379FB0D81B6925D  created: 2016-11-30  expires: 2018-11-30  usage: A   
[ultimate] (1)  Alice <alice@example.com>
[ unknown] (2). Alice <alice@example.org>

gpg> uid 1

pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C
                               trust: ultimate      validity: ultimate
sub  3072R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S
sub  3072R/0x76737ABEB92745D7  created: 2016-11-30  expires: 2018-11-30  usage: E
sub  3072R/0xE379FB0D81B6925D  created: 2016-11-30  expires: 2018-11-30  usage: A
[ultimate] (2)* Alice <alice@example.com>
[ultimate] (1). Alice <alice@example.org>

gpg> primary

You need a passphrase to unlock the secret key for
user: "Alice <alice@example.com>"
4096-bit RSA key, ID 0xD93D03C13478D580, created 2016-11-30

Enter passphrase: YourPassword

pub  4096R/0xD93D03C13478D580  created: 2016-11-30  expires: 2018-11-30  usage: C
                               trust: ultimate      validity: ultimate
sub  3072R/0x1ED73636975EC6DE  created: 2016-11-30  expires: 2018-11-30  usage: S
sub  3072R/0x76737ABEB92745D7  created: 2016-11-30  expires: 2018-11-30  usage: E
sub  3072R/0xE379FB0D81B6925D  created: 2016-11-30  expires: 2018-11-30  usage: A
[ultimate] (2)* Alice <alice@example.com>
[ultimate] (1)  Alice <alice@example.org>

gpg> save
```

Останній вихід після додавання ідентифікатора показує ключ із довірою «unknown», який зміниться на необмежений після збереження нового посвідчення за допомогою команди «save».

Позначення "." показує головний ідентифікатор для цього ключа. Як показано вище, після додавання другої особи, доданий ідентифікатор вибирається як основний. Але ми визначили основний ідентифікатор вручну шляхом вибору потрібного посвідчення за допомогою команди «uid», а потім команди «primary», щоб встановити цю особистість як первинну. Якщо ми додавали тільки пошту, головним записом буде запис з ім'ям.

## Експортуємо резервну копію публічного та секретного ключів

Щоб створити резервну копію ключів, експортуйте їх у файл. Експорт ключів виконується в два етапи, приватні ключі та секретні ключі експортуються окремо. 

```bash
$ gpg --homedir ./gnupg-test --export-secret-keys --armor --output secret-keys.gpg 0xD93D03C13478D580
$ gpg --homedir ./gnupg-test --export --armor --output public-keys.gpg 0xD93D03C13478D580
```

За допомогою першої команди всі секретні ключі (головні + підключі) експортуються в один файл. Друга команда експортує всі публічні ключі (головні + підключі) в інший файл. Ці файли можна використовувати для резервного копіювання створених ключів зберігши їх на окремий носій.

Щоб експортувати лише один конкретний підключ, ідентифікатор підключа можна вказати за допомогою «!» Знак оклику в кінці ID ключа вказує gpg експортувати лише цей конкретний підключ. Це доцільно зробити для підключів які будуть використовуватись на окремих пристроях. 

```bash
$ gpg --homedir ./gnupg-test --export-secret-subkeys --armor --output secret-subkey_sign.gpg 0x1ED73636975EC6DE!
```

Наведена вище команда експортує лише секретний ключ підключа для підпису. Довідкова сторінка gpg описує знак оклику в ідентифікаторі ключа так.

"Під час використання gpg знак оклику (!) може бути доданий, щоб примусово використовувати вказаний первинний або вторинний ключ, а не намагатися обчислити, який первинний або вторинний ключ використовувати." 

## Зв'язка ключів для щоденного використання

Для щоденного використання необхідно «вилучити» головний секретний ключ зі зв'язки щоб ризикувати ним. Таку зв'язку можна підготувати кількома шляхами.

### Варіант 1. Видалення секретного головного ключа

Експортуємо лише секретні підключі, видаляємо всі секретні частини ключів цього ключа з комплекту ключів (що включає не тільки головний ключ, але й підключі), а потім повторно імпортуючи лише секретні підключів залишаючи зв'язку без секретної частини головного ключа. 

```bash
$ gpg --homedir ./gnupg-test --export-secret-subkeys --armor --output secret-subkeys.gpg 0xD93D03C13478D580
```

Ця команда експортує всі секретні підключі заданого ідентифікатора ключа та збереже його у вказаному файлі. 

```bash
$ gpg --homedir ./gnupg-test --delete-secret-keys 0xD93D03C13478D580
gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


sec  4096R/0xD93D03C13478D580 2016-11-30 Alice <alice@example.org>

Delete this key from the keyring? (y/N) Y
This is a secret key! - really delete? (y/N) Y
```

GnuPG другої версії просить видаляти кожен секретний ключ/підключ. На цьому етапі можна прийняти операцію видалення головного ключа, але видалити підключі можна заборонити. Це призведе до повідомлення про помилку для операції видалення, про те що підключи не були видалені. Так можна видалити секретний головний ключ, але зберегти підключі, а отже, не буде потреби повторного імпорту секретних підключів.

Після виконання наведеної вище команди всі секретні головні ключі видаляються зі зв'язки. За допомогою наступної команди експортовані секретні підключі повторно імпортуються назад до зв'язки.

```bash
$ gpg --homedir ./gnupg-test --import ./gnupg-backup/secret-subkeys.gpg
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
$ gpg --homedir ./gnupg-test --export-secret-subkeys --armor --output secret-subkeys.gpg 0xD93D03C13478D580
```

Як і в першому варіанті, експорт лише з підключами має бути здійснений за допомогою команди вище. На додаток до наступних команд, я пропоную також скопіювати **gpg.conf**, який використовується в наборі ключів, щоб створити ключ для щоденного використання.

```bash
$ gpg --homedir ~/.gnupg --import gnupg-backup/public-keys.gpg
gpg: key 0xD93D03C13478D580: public key "Alice <alice@example.org>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
```

Наведену вище команду можна використовувати для імпорту публічного ключа до зв'язки, яку ви використовуєте щоденно. За допомогою наведеної нижче команди секретні підключі без головного секретного ключа можна імпортувати до щоденного використання ключів. «–homedir ~/.gnupg» можна опустити, якщо зв'язка знаходиться за замовчуванням, як у цьому прикладі. 

```bash
$ gpg --homedir ~/.gnupg --import gnupg-backup/secret-subkeys.gpg
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
$ gpg --homedir ./gnupg-test -K
./gnupg-test/secring.gpg
------------------------
sec#  4096R/0xD93D03C13478D580 2016-11-30 [expires: 2018-11-30]
      Key fingerprint = F8C8 1342 2A7F 7A3A 9027  E158 D93D 03C1 3478 D580
uid                            Alice <alice@example.com>
uid                            Alice <alice@example.org>
ssb   3072R/0x1ED73636975EC6DE 2016-11-30
      Key fingerprint = 292D 3E78 6B2E DBEA 1D10  02C8 1ED7 3636 975E C6DE
ssb   3072R/0x76737ABEB92745D7 2016-11-30
      Key fingerprint = 0C33 42E5 670A B099 8ED7  3E87 7673 7ABE B927 45D7
ssb   3072R/0xE379FB0D81B6925D 2016-11-30
      Key fingerprint = 7357 2158 947D BAFF A89F  4911 E379 FB0D 81B6 925D
```

Нарешті, зберігайте резервну копію секретних ключів, зокрема секретного головного ключа, і видаліть будь-який інший тимчасовий файл експорту секретних підключів, який більше не потрібен. Розгляньте можливість використання утиліти [shred](https://linux.die.net/man/1/shred), щоб безпечно видалити ці файли. 

## Перевірте створений ключ GnuPG

Як завжди, важливо перевірити результат. На щастя, є інструмент, який допоможе в цьому. hkt (hopenpgp-tools) забезпечують саме це. Утиліта не встановлюється у системі за замовчуванням, але може бути встановлена безпосередньо з репозиторію Ubuntu / LinuxMint. Здається наразі honpgp-tools недоступний у форматі rpm для RHEL або CentOS.

```bash
# Debian / Ubuntu
$ sudo apt-get install hopenpgp-tools

# RedHat / CentOS
# ... not available.
```

За допомогою hopenpgp-tools можна перевірити ключ для налаштування hopenpgp-tools. У наведеній нижче команді використовується «hkt», який є частиною hopenpgp-tools, щоб витягти публічний ключ із зв'язки ключів. Крім того, утиліта hopenpgp-tools під назвою «hokey» виконує перевірку ключових налаштування. Зелені розділи показують, де реалізується найкраща практика. Якщо налаштування не відповідає найкращій практиці, утиліта позначить його червоним.

```bash
$ hkt export-pubkeys --keyring gnupg-test/pubring.gpg 0xD93D03C13478D580 | hokey lint
hokey (hopenpgp-tools) 0.17
Copyright (C) 2012-2015  Clint Adams
hokey comes with ABSOLUTELY NO WARRANTY. This is free software, and you are welcome to redistribute it under certain conditions.
hkt (hopenpgp-tools) 0.17
Copyright (C) 2012-2015  Clint Adams
hkt comes with ABSOLUTELY NO WARRANTY. This is free software, and you are welcome to redistribute it under certain conditions.

Key has potential validity: good
Key has fingerprint: F8C8 1342 2A7F 7A3A 9027  E158 D93D 03C1 3478 D580
Checking to see if key is OpenPGPv4: V4
Checking to see if key is RSA or DSA (>= 2048-bit): RSA 4096
Checking user-ID- and user-attribute-related items:
  Alice <alice@example.com>:
    Self-sig hash algorithms: [SHA-512]
    Preferred hash algorithms: [SHA-512, SHA-384, SHA-256, SHA-224]
    Key expiration times: [1y11m29d81000s = Fri Nov 30 21:44:14 UTC 2018]
    Key usage flags: [[certify-keys]]
  Alice <alice@example.org>:
    Self-sig hash algorithms: [SHA-512]
    Preferred hash algorithms: [SHA-512, SHA-384, SHA-256, SHA-224]
    Key expiration times: [1y11m29d81000s = Fri Nov 30 21:44:14 UTC 2018]
    Key usage flags: [[certify-keys]]
```

Іншим способом перевірки згенерованого ключа GnuPG є pgpdump. Ця утиліта не інтерпретуватиме ключову інформацію та позначатиме будь-яку інформацію, яка вважається неприпустимою, але покаже вам необроблену ключову інформацію. З іншого боку, вона розкриває набагато більше деталей про сам ключ.

Утиліта pgpdump також не встановлена за замовчуванням, але її можна встановити безпосередньо зі сховищ дистрибутиву. На відміну вад утиліти «hopenpgp-tools», pgpdump доступний як rpm для RedHat і CentOS.

```bash
# Debian / Ubuntu
$ sudo apt install pgpdump

# RedHat / CentOS
$ yum install pgpdump
```

Програма pgpdump бере експорт секретного ключа, отриманого з резервної копії, і видає всю його інформацію у зрозумілому людям форматі. Наведена нижче команда показує згруповану інформацію для головного ключа та підключів.

Перший блок містить інформацію про секретний головний ключ. Виділена область в першому блоці показує налаштування s2k, визначені в gpg.conf.

```bash
$ gpg --homedir ./gnupg-test --export-secret-keys --armor 0xD93D03C13478D580 | pgpdump 
Old: Secret Key Packet(tag 5)(1862 bytes)
   Ver 4 - new
   Public key creation time - Wed Nov 30 22:44:14 CET 2016
   Pub alg - RSA Encrypt or Sign(pub 1)
   RSA n(4096 bits) - ...
   RSA e(17 bits) - ...
   Sym alg - AES with 256-bit key(sym 9)
   Iterated and salted string-to-key(s2k 3):
      Hash alg - SHA512(hash 10)
      Salt - 12 43 4f 59 74 01 a2 ff 
      Count - 1015808(coded count 159)
   IV - 2f d5 54 c1 1f f0 a6 87 a3 af 58 ef 69 47 ec 4f 
   Encrypted RSA d
   Encrypted RSA p
   Encrypted RSA q
   Encrypted RSA u
   Encrypted SHA1 hash
```

Наведене вище показує, що цей ключ - це RSA 4096. Перший блок показує ідентифікатор та сигнатуру ключа. Це ясно показує ідентичність підпису головного ключа. У підпису Hash алгоритм SHA512, а також переважають алгоритми, як визначено в pgp.conf, і можна знайти сам ключа.

```bash
Old: User ID Packet(tag 13)(25 bytes)
   User ID - Alice <alice@example.com>
Old: Signature Packet(tag 2)(573 bytes)
   Ver 4 - new
   Sig type - Positive certification of a User ID and Public Key packet(0x13).
   Pub alg - RSA Encrypt or Sign(pub 1)
   Hash alg - SHA512(hash 10)
   Hashed Sub: signature creation time(sub 2)(4 bytes)
      Time - Wed Nov 30 22:44:14 CET 2016
   Hashed Sub: key flags(sub 27)(1 bytes)
      Flag - This key may be used to certify other keys
   Hashed Sub: key expiration time(sub 9)(4 bytes)
      Time - Fri Nov 30 22:44:14 CET 2018
   Hashed Sub: preferred symmetric algorithms(sub 11)(4 bytes)
      Sym alg - AES with 256-bit key(sym 9)
      Sym alg - AES with 192-bit key(sym 8)
      Sym alg - AES with 128-bit key(sym 7)
      Sym alg - CAST5(sym 3)
   Hashed Sub: preferred hash algorithms(sub 21)(4 bytes)
      Hash alg - SHA512(hash 10)
      Hash alg - SHA384(hash 9)
      Hash alg - SHA256(hash 8)
      Hash alg - SHA224(hash 11)
   Hashed Sub: preferred compression algorithms(sub 22)(4 bytes)
      Comp alg - ZLIB <RFC1950>(comp 2)
      Comp alg - BZip2(comp 3)
      Comp alg - ZIP <RFC1951>(comp 1)
      Comp alg - Uncompressed(comp 0)
   Hashed Sub: features(sub 30)(1 bytes)
      Flag - Modification detection (packets 18 and 19)
   Hashed Sub: key server preferences(sub 23)(1 bytes)
      Flag - No-modify
   Sub: issuer key ID(sub 16)(8 bytes)
      Key ID - 0xD93D03C13478D580
   Hash left 2 bytes - b4 17 
   RSA m^d mod n(4096 bits) - ...
      -> PKCS-1
Old: User ID Packet(tag 13)(25 bytes)
   User ID - Alice <alice@example.org>
Old: Signature Packet(tag 2)(573 bytes)
   Ver 4 - new
   Sig type - Positive certification of a User ID and Public Key packet(0x13).
   Pub alg - RSA Encrypt or Sign(pub 1)
   Hash alg - SHA512(hash 10)
   Hashed Sub: signature creation time(sub 2)(4 bytes)
      Time - Wed Nov 30 22:44:28 CET 2016
   Hashed Sub: key flags(sub 27)(1 bytes)
      Flag - This key may be used to certify other keys
   Hashed Sub: key expiration time(sub 9)(4 bytes)
      Time - Fri Nov 30 22:44:14 CET 2018
   Hashed Sub: preferred symmetric algorithms(sub 11)(4 bytes)
      Sym alg - AES with 256-bit key(sym 9)
      Sym alg - AES with 192-bit key(sym 8)
      Sym alg - AES with 128-bit key(sym 7)
      Sym alg - CAST5(sym 3)
   Hashed Sub: preferred hash algorithms(sub 21)(4 bytes)
      Hash alg - SHA512(hash 10)
      Hash alg - SHA384(hash 9)
      Hash alg - SHA256(hash 8)
      Hash alg - SHA224(hash 11)
   Hashed Sub: preferred compression algorithms(sub 22)(4 bytes)
      Comp alg - ZLIB <RFC1950>(comp 2)
      Comp alg - BZip2(comp 3)
      Comp alg - ZIP <RFC1951>(comp 1)
      Comp alg - Uncompressed(comp 0)
   Hashed Sub: features(sub 30)(1 bytes)
      Flag - Modification detection (packets 18 and 19)
   Hashed Sub: key server preferences(sub 23)(1 bytes)
      Flag - No-modify
   Sub: issuer key ID(sub 16)(8 bytes)
      Key ID - 0xD93D03C13478D580
   Hash left 2 bytes - e5 1a 
   RSA m^d mod n(4096 bits) - ...
      -> PKCS-1
```

Після головного ключа та ідентифікаторів можна знайти секретні ключі. Перший – це ключ підпису розміром 3072 біти. Частина виводу з підписами показує два підписи, один з головного ключа і один підпис від самого себе (самопідпис). Сам підпис можна ідентифікувати за «ідентифікатором ключа емітента».

```bash
Old: Secret Subkey Packet(tag 7)(1414 bytes)
   Ver 4 - new
   Public key creation time - Wed Nov 30 22:44:21 CET 2016
   Pub alg - RSA Encrypt or Sign(pub 1)
   RSA n(3072 bits) - ...
   RSA e(17 bits) - ...
   Sym alg - AES with 256-bit key(sym 9)
   Iterated and salted string-to-key(s2k 3):
      Hash alg - SHA512(hash 10)
      Salt - 0f 7d 79 55 7c 4e 99 71 
      Count - 1015808(coded count 159)
   IV - f6 1f ba 7c d4 40 28 19 72 c5 12 0f 4f c1 a3 65 
   Encrypted RSA d
   Encrypted RSA p
   Encrypted RSA q
   Encrypted RSA u
   Encrypted SHA1 hash
Old: Signature Packet(tag 2)(964 bytes)
   Ver 4 - new
   Sig type - Subkey Binding Signature(0x18).
   Pub alg - RSA Encrypt or Sign(pub 1)
   Hash alg - SHA512(hash 10)
   Hashed Sub: signature creation time(sub 2)(4 bytes)
      Time - Wed Nov 30 22:44:21 CET 2016
   Hashed Sub: key flags(sub 27)(1 bytes)
      Flag - This key may be used to sign data
   Hashed Sub: key expiration time(sub 9)(4 bytes)
      Time - Fri Nov 30 22:44:21 CET 2018
   Sub: issuer key ID(sub 16)(8 bytes)
      Key ID - 0xD93D03C13478D580
   Sub: embedded signature(sub 32)(412 bytes)
   Ver 4 - new
   Sig type - Primary Key Binding Signature(0x19).
   Pub alg - RSA Encrypt or Sign(pub 1)
   Hash alg - SHA512(hash 10)
   Hashed Sub: signature creation time(sub 2)(4 bytes)
      Time - Wed Nov 30 22:44:21 CET 2016
   Sub: issuer key ID(sub 16)(8 bytes)
      Key ID - 0x1ED73636975EC6DE
   Hash left 2 bytes - 2c da 
   RSA m^d mod n(3071 bits) - ...
      -> PKCS-1
   Hash left 2 bytes - c0 7e 
   RSA m^d mod n(4095 bits) - ...
      -> PKCS-1
```

Ключ шифрування, а також ключ автентифікації показують лише один підпис головного ключа. Оскільки їхні можливості не включають підписання, вони не підписані самі собою.

```bash
Old: Secret Subkey Packet(tag 7)(1414 bytes)
   Ver 4 - new
   Public key creation time - Wed Nov 30 22:44:23 CET 2016
   Pub alg - RSA Encrypt or Sign(pub 1)
   RSA n(3072 bits) - ...
   RSA e(17 bits) - ...
   Sym alg - AES with 256-bit key(sym 9)
   Iterated and salted string-to-key(s2k 3):
      Hash alg - SHA512(hash 10)
      Salt - 5c 54 ae 09 9d d2 01 c6 
      Count - 1015808(coded count 159)
   IV - 6d 44 b1 68 d4 c5 94 41 c0 23 ea 92 bc 50 ce 68 
   Encrypted RSA d
   Encrypted RSA p
   Encrypted RSA q
   Encrypted RSA u
   Encrypted SHA1 hash
Old: Signature Packet(tag 2)(549 bytes)
   Ver 4 - new
   Sig type - Subkey Binding Signature(0x18).
   Pub alg - RSA Encrypt or Sign(pub 1)
   Hash alg - SHA512(hash 10)
   Hashed Sub: signature creation time(sub 2)(4 bytes)
      Time - Wed Nov 30 22:44:23 CET 2016
   Hashed Sub: key flags(sub 27)(1 bytes)
      Flag - This key may be used to encrypt communications
      Flag - This key may be used to encrypt storage
   Hashed Sub: key expiration time(sub 9)(4 bytes)
      Time - Fri Nov 30 22:44:23 CET 2018
   Sub: issuer key ID(sub 16)(8 bytes)
      Key ID - 0xD93D03C13478D580
   Hash left 2 bytes - fb 43 
   RSA m^d mod n(4096 bits) - ...
      -> PKCS-1
Old: Secret Subkey Packet(tag 7)(1414 bytes)
   Ver 4 - new
   Public key creation time - Wed Nov 30 22:44:26 CET 2016
   Pub alg - RSA Encrypt or Sign(pub 1)
   RSA n(3072 bits) - ...
   RSA e(17 bits) - ...
   Sym alg - AES with 256-bit key(sym 9)
   Iterated and salted string-to-key(s2k 3):
      Hash alg - SHA512(hash 10)
      Salt - a5 7a f7 44 d2 b7 3c 2d 
      Count - 1015808(coded count 159)
   IV - 25 3c aa 37 c9 8e 16 eb 12 b6 7d 04 36 a3 db 92 
   Encrypted RSA d
   Encrypted RSA p
   Encrypted RSA q
   Encrypted RSA u
   Encrypted SHA1 hash
Old: Signature Packet(tag 2)(549 bytes)
   Ver 4 - new
   Sig type - Subkey Binding Signature(0x18).
   Pub alg - RSA Encrypt or Sign(pub 1)
   Hash alg - SHA512(hash 10)
   Hashed Sub: signature creation time(sub 2)(4 bytes)
      Time - Wed Nov 30 22:44:26 CET 2016
   Hashed Sub: key flags(sub 27)(1 bytes)
      Flag - This key may be used for authentication
   Hashed Sub: key expiration time(sub 9)(4 bytes)
      Time - Fri Nov 30 22:44:26 CET 2018
   Sub: issuer key ID(sub 16)(8 bytes)
      Key ID - 0xD93D03C13478D580
   Hash left 2 bytes - 9d 8b 
   RSA m^d mod n(4095 bits) - ...
      -> PKCS-1
```

Навіть якщо використовувати gpgdump не так зручно, як перевіряти ключ за допомогою hopenpgp-tools, є інші деталі, які можуть бути цікавими. Крім того, для RedHat / CentOS та інших дистрибутивах на основі rpm він пропонує хорошу альтернативу honpgp-tools.

Адаптація і доповнення статті ["Create GnuPG key with sub-keys to sign, encrypt, authenticate"](https://blog.tinned-software.net/create-gnupg-key-with-sub-keys-to-sign-encrypt-authenticate/)
