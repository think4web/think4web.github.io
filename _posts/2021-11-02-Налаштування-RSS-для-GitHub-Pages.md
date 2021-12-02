---
published: false
---

## Підключаємо плагін для генерації фіда

Перевіряємо або додаємо у `_config.yml` наступне:

```yml
gems:
  - jekyll-feed
name: %your_name%
description: %your_blog_description%
```
GitHub Pages підтримують [наступні плагіни](https://pages.github.com/versions/) але вони вимкнені за замовчуванням. [jekyll-feed](https://github.com/jekyll/jekyll-feed) один з них. Щоб увімкнути плагін достатньо додати його у блок `gems`. І після публікації нових налаштувань сайт сгенерую сторінку `feed.xml`, яка стане доступною за адресою `%website_name%/feed.xml`.

Властивості `name` та  `description` не обов'язкові. Додаткові налаштування можна подивитися у репозиторії [jekyll-feed](https://github.com/jekyll/jekyll-feed).

## Налаштовуємо шаблон HTML для RSS-читачів

Перевіряємо або додаємо у шаблон HTML `_layouts/default.html` наступний рядок:
```html
<link rel="alternate" type="application/rss+xml" title="{{ site.name }} - {{ site.description }}" href="{{ site.baseurl }}/feed.xml" />
```
`{{ site.name }}` формується з `name` вказаному в `_config.yml`.

Це допоможе підписатися на блог за допомогою RSS-читачів.

### Додаємо посилання на RSS-канал

Додаємо у шаблон сторінки `_layouts/default.html` або у налаштування сайту `_config.yml` посилання на фід на усі сторінки блогу:
```html
<a href="{{ site.baseurl }}/feed.xml">RSS</a>
```
Все готово, а сам процес налаштування займає кілька хвилин.

---
За публікацією [Dzhavat Ushev](https://dzhavat.github.io/2020/01/19/adding-an-rss-feed-to-github-pages.html).
