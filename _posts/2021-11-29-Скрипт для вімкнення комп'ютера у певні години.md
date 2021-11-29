---
published: true
---
## Скрипт для вімкнення комп'ютера у певні години

```bash
# The code belou will stop me from using computer after 23:00 to 5:00 
  
time=$(date '+%H')
if 
	[ "$time" >  "22:00" ] || [ "$time" < "05:00" ];
then systemctl suspend
fi
```
