 Quit/run global protect
```
launchctl unload /Library/LaunchAgents/com.paloaltonetworks.gp.pangp*
launchctl load /Library/LaunchAgents/com.paloaltonetworks.gp.pangp*
```

Сделал ADSDEV-1369 в post-processor (4d)
- добавил 2 функции для 2х ручек и удалил одну старую
- сделал тесты
- Вика добавила комменты - сделал правки в тестах для ADSDEV-1369 в post-processor
Перенес скрипт agent_info.lua для post-processing-rules-test (1d)
Написал продьюсера который отправляет сообщение в кафку с вызовом этой функции, запрос уходит без ошибок но в логах ничего (1d)
Сделал отправку через клиента кафки - ошибка появилась (1d)
Делаю изменения в post-processor для подтягивания изменений в post-processing-rules-test на стенде (2d)
Сделал правку для сурка (1d)