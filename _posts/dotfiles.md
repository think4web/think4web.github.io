 ---
published: false
layout: post
title:
tags: git linux 
---


```bash
mkdir .dotfiles

# Створюємо новий репозиторій і називаємо його точно так само як назвали папку, після першого push репозиторій можна буде перейменувати
git init --bare $HOME/.dotfiles # Створюємо відкритий репозиторій
alias dotfiles='/usr/bin/git --git-dir=$HOME/dotfiles/ --work-tree=$HOME' # (add this alias to .bashrc)
bash
dotfiles config --local status.showUntrackedFiles no
```

У різних репозиторіях можна збирати конфігураційні файли для різних оболонок або систем.

Джерело: [DistroTube:Git Bare Repository](https://www.youtube.com/watch?v=tBoLDpTWVOM)
