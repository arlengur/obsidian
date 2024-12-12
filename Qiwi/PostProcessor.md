Схема работы:
- ДМ отправляет сообщение в кафку, 
- ПП его читает и запускает скрипт в Луа, 
- Луа вызывает ПП через af_api, 
- ПП делает запрос в Фасад и возвращает ответ в Луа.

```
{
  "commandId": "123",
  "sourceOrgId": 0,
  "sourceMessageTypeId": "20",
  "sourceMessageId": "234",
  "command": "agent_info",
  "arguments": [
	1
  ],
  "scheduledIsoTimestamp": null
}
```

Отправить сообщение в кафку для проверки post-processing-test (креды взять из https://passwork.qiwi.com/enter#/login для deсision-maker-test так как он пишет в кафку) скопировать ssl.truststore.password и ssl.keystore.password и скачать `*.jks` для ssl.truststore.location и ssl.keystore.location

получится так
```hocon
post-processing {  
    writer {  
        topic = ads_decision_maker_postprocessor_v1  
        kafka-clients {  
            bootstrap.servers = "kafka-ao-broker-test-m1-01.qiwi.com:9093,kafka-ao-broker-test-m1-02.qiwi.com:9093,kafka-ao-broker-test-dl-01.qiwi.com:9093,kafka-ao-broker-test-dl-02.qiwi.com:9093"  
            security.protocol = SSL  
            ssl.truststore.location = /Users/a.galin/repo/scala-kafka/src/main/resources/client.ao-ads-decision-maker-test.truststore.jks  
            ssl.truststore.password = A6keyCx7D5hWjfZMLb  
            ssl.keystore.location = /Users/a.galin/repo/scala-kafka/src/main/resources/client.ao-ads-decision-maker-test.keystore.jks  
            ssl.keystore.password = 8lnuRhrGEM38HWj9k9  
            ssl.key.password = 8lnuRhrGEM38HWj9k9  
            ssl.protocol = TLSv1.2  
        }  
    }  
}
```

Проект https://github.com/arlengur/scala-kafka.git

Команда в кафке
```
kafka-console-producer.sh --topic decision_maker_postprocessing_v1 --bootstrap-server kafka-ao-broker-test-m1-01.qiwi.com:9093 --producer.config /Users/a.galin/repo/scala-kafka/src/main/resources/ao-kafka-ssl-config-dm.properties
```

ao-kafka-ssl-config-dm.properties:
```
bootstrap.servers=kafka-ao-broker-test-m1-01.qiwi.com:9093  
security.protocol=SSL  
ssl.truststore.location=...jks  
ssl.truststore.password=...  
ssl.keystore.location=...jks  
ssl.keystore.password=...
ssl.key.password=...
ssl.protocol=TLSv1.2
```

Посмотреть можно в 
- https://qoq-ao-test.qiwi.com/
	- там open и   ao-ads-post-processor test
- https://grafana.qiwi.com/
	- IRIS - services - ao-ads-post-processor
- https://dashboard.testing.qiwi.com/
	- name space: ao-ads
	- pods
	- тут можно посмотреть логи

Сваггер апи фасада: 
https://ao-fraud-automatization-api-facade.testing.qiwi.com/api/v1.0/specification

тест
{"commandId":"124","sourceOrgId":0,"sourceMessageTypeId":"20","sourceMessageId":"234","command":"get_agent_info_by_id1","arguments":[1],"scheduledIsoTimestamp":null}

|function get_qd_manager_email_by_agt_id(agtId)  
|  local manager_email = af_api.get_qd_manager_email_by_agt_id(agtId)  
|  if manager_email == "name@mail.ru" then  
|   logger.info(agt_full_name .. agt_distributor_id)  
|  else  
|   error("Wrong manager prs_email=" .. manager_email)  
|  end  
|end