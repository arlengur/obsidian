ExtractorService - WatcherManager - WatcherFactory

watcher - proc - chain

Watcher 
ExtractorService - WatcherManager (WatcherFactory)


Вопросы
- SystemType, SystemId ??
- validateContentType - ContentType from HpiumDictionaryItem

 call-type-id (https://confluence.mts.ru/pages/viewpage.action?pageId=1621479998)
- 12 - исходящий
- 13 - входящий

Описание полей https://confluence.mts.ru/pages/viewpage.action?pageId=1349643576 (интерфейсное соглашение с HP IUM 1.27)
- imsi - международный идентификатор мобильного абонента
- imei - международный идентификатор мобильного оборудования
- inboundBunch - входящая трангруппа (название соединительной линии между двумя коммутаторами). Если аб мегафон звонит в мтс, то два коммутатора соединены по транковой группе. Если звонок исходящий О - то и трангруппа исходящая.
- TermCause - причина завершения звонка
- Supplement Service - дополнительный вид обслуживания
- telco - регион возникновения события
- SwitchId - это коммутатор обработавший вызов
- Roaming Partner - когда абонент за границей в поле Roaming Partner стоит название оператора к которому подключался абонент (TAP файлы роаминга)

- Если direction(0) и есть inInfo - есть если звонок исходящий то берем данные у абонента кто звонит
- Если direction(1) и есть outInfo - если звонок входящий то берем данные у абонента кому звонят

Прочитать сообщения из кафки
```
kafka-console-consumer.sh --bootstrap-server sorm-test-queue01.ural.mts.ru:6667 --topic ag_mobile_connections --from-beginning --max-messages 5
```

В интерпретации HP IUM, экстрактора и публикатора 
- О - исходящий вызов, но в трифт модели и норси - это in-info
- I - входящий вызов, но в трифт и норси заносится в поле out-info

## Docker

### Запустить образ оракл
`docker run -p 1521:1521 --name ext 25ca131c492f`

### Остановить образ оракл
`docker stop ext`
`docker rm ext`


125 138