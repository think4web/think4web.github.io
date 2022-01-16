---
published: true
layout: post
title: Детальна перевірка ключів GnuPG 
author: think4web
discription: Дуже детальна перевірка створених GnuPG ключів.
tags: gpg
---

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

Переклад і доповнення статті: ["Create GnuPG key with sub-keys to sign, encrypt, authenticate"](https://blog.tinned-software.net/create-gnupg-key-with-sub-keys-to-sign-encrypt-authenticate/)
