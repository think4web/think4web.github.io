---
published: true
layout: post
title: KeePassXC - вільний менеджер паролів 
author: think4web
discription: Вільний менеджер паролів з відкритим кодом.
tags: software android totp keepassxc
---

[KeePassXC](https://keepassxc.org/) - менеджер паролів з відкритим кодом. Паролі зберігає локально у шифрованій базі. Вміє генерувати одноразові коди TOTP і дозволяє прикріплювати файли, тобто дозвляє крім паролів безпечно зберігати документи. Можна задати термін через який мине строк придатності паролю (тобто дату коли його пора змінити). Має додаток для браузеру [Firefox](https://addons.mozilla.org/en-US/firefox/addon/keepassxc-browser/) для автоматичного введення паролів та версію для [Android](https://f-droid.org/uk/packages/com.kunzisoft.keepass.libre/) з тією ж функцією. База паролів між десктопом і смартфоном чудова синхронізується. Хоча десктопна версія менш зручна у користуванні, вона розуміє [гарячі клавіші](file:///usr/share/keepassxc/docs/KeePassXC_KeyboardShortcuts.html), що прискорює роботу. Вбудований генератор паролів можна використовувати без створення нового запису.

## Налаштування в LibreWolf

Створити у .librewolf папку native-messaging-hosts і у налаштуваннях KeePassX Tools -> Settings -> Browser Integration -> Advanced вписати у розташуваннях параметрів шлях до цієї папки ```~/.librewolf/native-messaging-hosts```. KeePassX попросить вибрати ідентифікатор пристрою і буде працювати як звичайно. 
