---
published: true
layout: post
title: Встановлення і налаштування Firefox 
author: think4web
discription: Як встановити і налаштувати Firefox на GNU/Linux для безпечного відвідування сайтів в інтернеті.
tags: firefox security
---

Встановлюємо ```firefox-esr```. Через термінал створюємо новий профіль:

```bash
firefox --no-remote -ProfileManager
```

Відкриваємо Firefox і переходимо на [spyware.neocities.org](https://spyware.neocities.org/guides/firefox.html) і виконуємо усе по інтрукції.

У налаштуваннях браузеру:
- прибираємо усі пошукові системи крім DuckDuckGo;
- "Безпека і приватність":
  - вимикаємо запам'ятовування логінів і паролів;
  - не запам'ятовувати історію браузеру;
  - не приймати cookies від третіх осіб;
  - зберігати дані сайтів до закриття програми.

Встановлюємо наступні додатки:
- [**HTTPS Everywhere**](https://addons.mozilla.org/en-US/firefox/addon/https-everywhere/) - примусово завантажує сторінки через HTTPS якщо на них є сертифікати безпеки;
- [**Ublock Origin**](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/) - блокувальник реклами;
- [**uMatrix**](https://addons.mozilla.org/en-US/firefox/addon/umatrix/) - блокує рекламу на рівні домену; 
- [**NoScript**](https://addons.mozilla.org/en-US/firefox/addon/noscript/) - блокує скрипти;
- [**Cookie AutoDelete**](https://addons.mozilla.org/en-US/firefox/addon/cookie-autodelete/) - автовидалення cockie після закриття сайту.
