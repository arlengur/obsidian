Документация 
- https://confluence.mts.ru/pages/viewpage.action?pageId=1425697154
Gitlab
- https://gitlab.services.mts.ru/dsorm/dwhs/dwhs-publisher
Repo
- https://artifacts.services.mts.ru/#browse/browse:dwhs-maven

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




Stage 0 contains a task of very large size (2109 KiB). The maximum recommended task size is 1000 KiB.

```scala
object GroupRowMessage {
    def apply(commId: Long, part: Int, messages: Iterable[RowMessage]): GroupRowMessage = {
        GroupRowMessage(Fsm.grouped, commId.toInt, part, messages.map(_.row))
    }
    def apply(commId: Int, messages: Iterable[RowMessage], maxPartSize: Int): Iterator[GroupRowMessage] = {
        messages
            .sliding(maxPartSize, maxPartSize)
            .zipWithIndex
            .map { case (messages, part) =>
                GroupRowMessage(Fsm.grouped, commId, part, messages.map(_.row))
            }
    }
}

trait GroupDsl {  
  implicit class GroupRowMessageOps(rdd: RDD[RowMessage]) extends Serializable {  
    def group(commIdIndex: Int, maxPartSize: Int): RDD[GroupRowMessage] = rdd  
      .groupBy(_.row.getAs[Int](commIdIndex))  
      .zipWithIndex  
      .map { case ((i, msg), commId) => (commId, msg, i / maxPartSize) }  
      .groupBy { case (commId, _, part) => (commId, part) }  
      .map { case ((commId, part), messages) =>  
        GroupRowMessage(commId, part, messages.flatMap{ case (_, mssages, _) => mssages})  
      }  
  }  
}
```

Spent time is 1.396s

```
package ru.mts.dsorm.dwhs.publisher.flow.dsl  
  
import org.apache.spark.rdd.RDD  
import ru.mts.dsorm.dwhs.publisher.flow.acc.GroupedDbAppAcc  
import ru.mts.dsorm.dwhs.publisher.flow.adapter.DbAdapter  
import ru.mts.dsorm.dwhs.publisher.flow.message.{GroupRowMessage, StatMessage}  
  
trait StoreDsl {  
    implicit class StoreDslOps(rdd: RDD[GroupRowMessage]) extends Serializable {  
        def store(dbAdapter: => DbAdapter[Int], acc: GroupedDbAppAcc): RDD[StatMessage] = {  
            println(">>>" + rdd.partitions.length)  
  
            rdd  
                    .mapPartitions(messages => if (messages.isEmpty) Iterator.empty else {  
                        val adapter = dbAdapter  
  
                        messages.map { message =>  
                            val stat = adapter.writeGroup(message.commId, message.rows.iterator)  
  
                            acc.groups.add(1)  
                            acc.tables.add(stat.numTables)  
                            acc.rows.add(stat.numRows)  
                            acc.minInGroup.add(stat.numRows)  
                            acc.maxInGroup.add(stat.numRows)  
                            acc.groupToRows.add((message.commId, stat.numRows))  
  
                            StatMessage(message, stat)  
                        }  
                    })  
                    .foreachPartition(i => println(i.size))  
  
            rdd  
                .mapPartitions(messages => if (messages.isEmpty) Iterator.empty else {  
                    val adapter = dbAdapter  
  
                    messages.map { message =>  
                        val stat = adapter.writeGroup(message.commId, message.rows.iterator)  
  
                        acc.groups.add(1)  
                        acc.tables.add(stat.numTables)  
                        acc.rows.add(stat.numRows)  
                        acc.minInGroup.add(stat.numRows)  
                        acc.maxInGroup.add(stat.numRows)  
                        acc.groupToRows.add((message.commId, stat.numRows))  
  
                        StatMessage(message, stat)  
                    }  
                })  
  
    }  
    }  
}
```

```
(0 to 8).map(i =>  
GroupRowMessage(State(Statuses.Grouped),i,Seq.fill(3)(i).map(i => ArrayRow(Array(i)))))
```

```
package ru.mts.dsorm.dwhs.publisher.flow.dsl  
  
import org.junit.runner.RunWith  
import org.scalatestplus.junit.JUnitRunner  
import ru.mts.dsorm.dwhs.publisher.core.fsm.State  
import ru.mts.dsorm.dwhs.publisher.flow.LocalSparkContextSpec  
import ru.mts.dsorm.dwhs.publisher.flow.adapter.ArrayRow  
import ru.mts.dsorm.dwhs.publisher.flow.fsm.Fsm.Statuses  
import ru.mts.dsorm.dwhs.publisher.flow.message.{GroupRowMessage, RecordMessage, RowMessage}  
  
@RunWith(classOf[JUnitRunner])  
class GroupDslSpec extends LocalSparkContextSpec {  
    "GroupDsl.group" should "return RDD of GroupRowMessage" in withSparkContext { sc =>  
        val mapper: Int => ArrayRow = i => ArrayRow(Array(i))  
        val data = Seq.tabulate(9,3)((i, _)=>i).flatten.map(i => RowMessage(RecordMessage(i), mapper))  
  
        val result = sc.parallelize(data).group(0, 1).collect()  
        val result2 = sc.parallelize(data)  
                .groupBy(_.row.getAs[Int](0))  
                .flatMap { case (commId, messages) =>  
                    messages.grouped(1).map(i => GroupRowMessage(commId, i)) }  
                .repartition(8)  
                .foreachPartition(i => println(i.size))  
  
  
//        result.length shouldBe 1  
//        result.head.state shouldBe State(Statuses.Grouped)  
//        result.head.commId shouldBe 1  
//        result.head.part shouldBe 0  
//        val adapterRows = result.head.rows.toList  
//        adapterRows.length shouldBe 3  
//        adapterRows.foreach(row => row.get(0) shouldBe 1)  
    }  
}
```