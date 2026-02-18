Документация 
- https://confluence.mts.ru/pages/viewpage.action?pageId=1425697154
Repo
- https://gitlab.services.mts.ru/dsorm/dwhs/dwhs-publisher

Для работы Gradle необходимо добавить сертификаты:
- Скачать https://pki.mts.ru/ сертификаты 
	- root.crt
	- class2root.crt
	- class3root.crt
	- WinCAG2.crt
- и установить их командами типа
```
keytool -importcert -file class3root.crt -keystore /Library/Java/JavaVirtualMachines/jdk-17.0.12.jdk/Contents/Home/lib/security/cacerts -alias class3root -storepass changeit -noprompt
```

Добавить строки в переменные среды (пр. .zshrc)
```
export ORG_GRADLE_PROJECT_ARTIFACTORY_USER=<USER>
export ORG_GRADLE_PROJECT_ARTIFACTORY_TOKEN=<PASSWORD>
export ORG_GRADLE_PROJECT_ARTIFACTORY_URL=https://artifactory.mts.ru/artifactory/
```

Создать свой топик http://mediation-test-elk01.ural.mts.ru:9000/ кластер sorm-test создать топик ag_mobile_connections с Partitions = 1, Replication Factor = 1

3626 - 1480 - 2035

colorit-nn@mail.ru

Добрый день, мне нужно напечатать флаера на формате а4, в цветном глянце (170гр).
4 набора:
- Forum of brahmachary 2025 – 25шт
- Tour of brahmachary 2025 – 12шт
- Бацйкал 20шт
- Домбай 40шт

yar-input-02.msk.mts.ru (10.233.212.33

## Настроить запуск в idea

VM ops
```
--add-opens=java.base/java.lang=ALL-UNNAMED
--add-opens=java.base/java.lang.invoke=ALL-UNNAMED
--add-opens=java.base/java.lang.reflect=ALL-UNNAMED
--add-opens=java.base/java.io=ALL-UNNAMED
--add-opens=java.base/java.net=ALL-UNNAMED
--add-opens=java.base/java.nio=ALL-UNNAMED
--add-opens=java.base/java.util=ALL-UNNAMED
--add-opens=java.base/java.util.concurrent=ALL-UNNAMED
--add-opens=java.base/java.util.concurrent.atomic=ALL-UNNAMED
--add-opens=java.base/sun.nio.ch=ALL-UNNAMED
--add-opens=java.base/sun.nio.cs=ALL-UNNAMED
--add-opens=java.base/sun.security.action=ALL-UNNAMED
--add-opens=java.base/sun.util.calendar=ALL-UNNAMED
```

-cp dwhs-publisher.main

Путь до main
`ru.mts.dsorm.dwhs.publisher.stream.mobile.MobileStreamPublisher`

Working directory
`/Users/agalin/repo/dwhs-publisher`

Аргументы
```
--spark-app-name
"MobileFlow"
--spark-master-url
"local[1]"
--spark-batch-duration
5
--spark-options
"spark.streaming.kafka.maxRatePerPartition=10,spark.serializer=org.apache.spark.serializer.KryoSerializer,spark.driver.bindAddress=127.0.0.1"
--spark-metrics
"spark.metrics.conf.driver.source.application.class=ru.mts.dsorm.dwhs.publisher.flow.acc.GroupedDbAppSource,spark.metrics.conf.driver.sink.prometheusServlet.class=org.apache.spark.metrics.sink.PrometheusServlet,spark.metrics.conf.driver.sink.prometheusServlet.path=/metrics/prometheus"
--kafka-bootstrap-servers
"sorm-test-queue01.ural.mts.ru:6667,sorm-test-queue02.ural.mts.ru:6667"
--kafka-topics
"ag_mobile_connections"
--kafka-group-id
"mobile_publisher_1"
--kafka-value-deserializer-class
"ru.mts.dsorm.dwhs.publisher.flow.serde.TDwhsMobileRecordDeserializer"
--kafka-options
"auto.offset.reset=latest,enable.auto.commit=false"
--db-url
"jdbc:oracle:thin:@//sorm-test-db01.ural.mts.ru:1521/sorm3tst"
--db-user
"dwhs_load_mob"
--db-password
"dwhs_load_mob"
--db-schema
"DWHS_LOAD_MOB"
--db-batch-size
1000
--db-table-size
1000
--db-idle-open-table-sec
30
```

Паблишер создает новые таблицы в БД: Schemas/DWHS_LOAD_MOB/Tables

 Подключиться к БД можно DBeaver: создать соединение, выбрать oracle, connection type - custom, ввести url, login, pass и сделать Test connection.


```
java -jar C:\Users\dngonch1\ws\git\dwhs\dwhs-extractor-app-1.3.0\dwhs-extractor-1.3.0-boot.jar --spring.config.location=C:\Users\dngonch1\ws\git\dwhs\dwhs-extractor-app-1.3.0\config\application.yml
```

Посмотреть метрики
https://confluence.mts.ru/pages/viewpage.action?pageId=1447450793

Локально:
localhost:4040/metrics/prometheus/

metrics_local_1750074522050_driver_app_groups_Count{type="counters"} 2
metrics_local_1750074522050_driver_app_groups_maxInGroup_Count{type="counters"} 122
metrics_local_1750074522050_driver_app_groups_minInGroup_Count{type="counters"} 122
metrics_local_1750074522050_driver_app_rows_Count{type="counters"} 122
metrics_local_1750074522050_driver_app_tables_Count{type="counters"} 2


RanapTestSuite
EnrichDslSpec
StoreDslSpec
SipTestSuite
SipiTestSuite
BaseScenariosSpec

11-12
7шт
15.00-16.00
