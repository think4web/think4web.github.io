# [think4web.github.io](https://think4web.github.io)
![Created Badge](https://badges.pufler.dev/created/think4web/dotfiles?color=blue)
![Updated Badge](https://badges.pufler.dev/updated/think4web/dotfiles?color=blue)
![Visits Badge](https://badges.pufler.dev/visits/think4web/dotfiles?color=blue)

## TODO
- [x] Add RSS
- [ ] Add style for code block

## Plugins

- **purify.min.js** - скрипт дає можливість підключти коментарі з Fediverse. Поки що його не задіяв.

## Theme

Для відключення капіталізації заголовків статей у `_layouts/post.html` змінив `{{ post.title }}` на `{{ page.title | downcase | capitalize }}` і у `index.html` змінив `{{ post.title }}` на `{{ post.title | downcase | capitalize }}`
