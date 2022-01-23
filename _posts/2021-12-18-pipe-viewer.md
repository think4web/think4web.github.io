---
published: true
layout: post
title: pipe-viewer - відтворення відео з YouTube
author: think4web
discription: Встановлення і налаштування pipe-viewer.
tags: software youtube pipe-verwer
---

[pipe-viewer](https://github.com/trizen/pipe-viewer) - це легка та багатофункціональна програма для відео з YouTube.

Програма парсить результати безпосередньо з YouTube, але може використовувати будь який інстанс Invidious як запасний варіант, що є більш надійним методом.

## Встановлення

Встановимо залежності:
```bash
sudo apt install git libwww-perl liblwp-protocol-https-per libdata-dump-perl
```

Завантажимо джерельний код з git і встановимо програму:
```bash
git clone https://github.com/trizen/pipe-viewer
cd pipe-viewer
perl Build.PL
./Build install
```

Запустимо програму ```pipe-viewer```.

## Приклади використання

Пошук по ключу:
```bash
pipe-viewer "<KEYWORD(S)>"
```

Або запустіть pipe-viewer і потім шукайте те що треба.

### Приклади опцій

Щоб побачити усі опції наберіть ```--help```:
- ```-uf, --favorites=<USER>``` - усі вподобані відео певним користувачем;
- ```-uv, --uploads=<USER>``` - усі відео завантажени певним користувачем;
- ```-a, --author=<USER>``` - шукати відео певного користувача;
- ```-rv, --related=<URL>``` - показати відео пов'язані з певним відео;
- ```--comments <URL>``` - показати коментарі до певного відео;
- ```-sp, --search-pl <KEYWORD>``` - шукати у плейлісту.

Фільтри:
- ```--order=view_count``` - сортувати за кількістю переглядів;
- ```--duration=<short/long>``` - сортування за тривалістю: короткі/довгі;
- ```--resolution=<VALUE>``` - роздільна здатність: best, 2160p, 1440p, 1080p, 720p, 480p, 360p, 240p, 144p, audio;
- ```--hd!``` - шукає відео якістю принаймні 720p.

Опції плеєру:
- ```-A``` - програти усі результати;
- ```-n, --novideo``` - тільки аудіо.

Переклад: [How-to install pipe-viewer in Debian](https://hund.tty1.se/2021/11/14/how-to-install-pipe-viewer-in-debian.html)
