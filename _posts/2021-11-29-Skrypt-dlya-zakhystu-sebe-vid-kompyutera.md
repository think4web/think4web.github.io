---
published: true
layout: post
title: Скрипт для захисту себе від комп'ютера
author: think4web
discription: Скрипт невблаганно і автоматично вимикає комп'ютер між 23 і 5 годинами що дає змогу поспати. 
tags: script
---

Скрипт невблаганно і автоматично вимикає (`systemctl suspend`) комп'ютер між 23 і 5 годинами що дає змогу поспати. 

```bash
#!/bin/sh

# The code belou will stop me from using computer after 23:00 to 5:00 
  
time=$(date '+%H')
if 
	[ "$time" -gt  22 ] || [ "$time" -lt 5 ];
then systemctl suspend
fi
```
Скрипт додав у `cron` для запуску кожну хвилину:
 ```
 * * * * * /file_path/script.sh
 ```
Комп'ютер вимикається, але при бажанні його можна розблокувати щоб через хвилину знову запустився скрипт і знову його вимкнув.
