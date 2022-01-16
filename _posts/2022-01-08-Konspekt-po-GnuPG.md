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
[ultimate] (1). Alice <alice@example.com>
gpg> **setpref SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed**
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
[ultimate] (1). Alice
gpg> save
```


## Клієнти

[OpenKeychain](https://f-droid.org/packages/org.sufficientlysecure.keychain/)

Переклад і доповнення статтей: 
- ["Create GnuPG key with sub-keys to sign, encrypt, authenticate"](https://blog.tinned-software.net/create-gnupg-key-with-sub-keys-to-sign-encrypt-authenticate/)
