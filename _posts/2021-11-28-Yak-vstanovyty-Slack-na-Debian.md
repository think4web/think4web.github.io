---
published: true
layout: post
title: Як встановити Slack на Debian
author: think4web
discription: Як встановити Slack пакетом .deb на Debian
tags: linux
---

На офіційному сайті [Slack](https://slack.com/) важко щось знайти і програма поширюється у .rpm, що мені не зручно. Тому ось інструкція як встановити Slack нормальним .deb пакетом.

Завантажуємо Slack:

```bash
wget https://downloads.slack-edge.com/linux_releases/slack-desktop-4.0.2-amd64.deb
```

Встановлюємо:

```bash
sudo dpkg -i slack-desktop-4.0.2-amd64.deb
```

Все готово, можна запускати Slack.
