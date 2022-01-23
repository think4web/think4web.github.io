---
published: true
layout: post
title: google-drive-ocamlfuse - клієнт Google Drive для Linux
author: think4web
discription: Програма для синхронізації Google Drive з директорією у Linux
tags: software google-drive-ocamlfuse 
---

[google-drive-ocamlfuse](https://github.com/astrada/google-drive-ocamlfuse) - програма для синхронізації з Google Drive.

Щоб примонтувати папку:

```bash
google-drive-ocamlfuse -o nonempty ~/%папка_для_синхронізації%/
```

Відмонтувати папку:

```bash
fusermount -u ~/%папка_для_синхронізації%
```
