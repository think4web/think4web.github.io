---
published: true
layout: post
title: Як зберігати файли конфігурації на GitHub
author: think4web
discription: Як зберігати файли конфігурації системи у репозиторії на GitHub.
tags: git linux 
---

Створюємо новий репозиторій і називаємо його точно так само як назвали папку, після першого push репозиторій можна буде перейменувати

```bash
git init --bare $HOME/.dotfiles # Створюємо відкритий репозиторій
alias dotfiles='/usr/bin/git --git-dir=$HOME/dotfiles/ --work-tree=$HOME' # додаємо новий alias у наш .bashrc
bash # перечитуємо конфіг
dotfiles config --local status.showUntrackedFiles no # не показувати status файлів що не були додані
```

Тепер щоб додати новий файл конфігу достатньо виконати ```dotfiles add`` і відправити файл у репозиторій.Так само легко можна отримати файли назад щоб відноситися. Маємо усі переваги git без зайвих рухів. 

У різних репозиторіях можна збирати конфігураційні файли для різних оболонок або систем.

Переклад [DistroTube:Git Bare Repository](https://www.youtube.com/watch?v=tBoLDpTWVOM)
