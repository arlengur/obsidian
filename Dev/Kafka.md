Это система, реализующая распределенный реплицируемый лог сообщений
- Распределенный - лог (топик) разбит на несколько секций (партиций), которые лежат на разных машинах
- Реплицируемый - логи хранятся в нескольких копиях, на случай отказов оборудования
- Лог - упорядоченная последовательность сообщений

Состав сообщения
- Ключ
	- опционален
	- массив байт
- Заголовки (для передачи мета информации)
	- опционален
	- набор пар ключ-значение (ключ - строка, значение - массив байт)
- Тело
	- массив байт

Протоколы в теле сообщения
- JSON
- XML
- Protobuf
- Thrift
- Avro

Топик - способ организации сообщений в Kafka (аналог таблицы в БД). Чаще всего это набор событий одного типа (customerCreated, notificationSent и тд), но в него также можно записывать сообщения (события) разных типов (customerCreated, customerInvoicePaid и тд), относящихся к одному бизнес процессу, это дает гарантии порядка. 
Слишком большое количество топиков (>1000) и партиций в одном брокере приводит к снижению производительности.

Топик - это набор неструктуированных сообщений (без схемы). Для kafka сообщение это просто набор байт поверх которых можно использовать разные протоколы. Для работы со схемой используют внешние инструменты SchemaRegistry и StreamProcessing (KTable, KStream)

Топик может иметь несколько отправителей и получателей

Принцип работы топика (это как запись в файл)
- отправитель добавляет сообщение в распределенный лог сообщений
- получатель (подписчик) читает из этого лога непрочтенные с прошлого раза записи (но может и любые другие)

Offset (номер сообщения в топике (партиции) которое потребитель прочитал)
- Каждый получатель (consumer) хранит у себя offset - указатель на сообщение, которое он прочитал последним
- может храниться на стороне получателя (в бд) или кафки (в специальном топике)

Commit offset - последняя обработанная запись - нужна чтобы понимать какие сообщения получатель обработал. Потребитель отправляет commit offset после того как обработал сообщение, если offset'ы хранятся на кафка. Периодичность его отправления определяет получателем (по умолчанию auto_commit=true, тогда получатели коммитят последний полученный из poll offset раз в 5с). Если получатель случайно упадет, то информация об обработанных сообщениях будет потеряна и сообщения будут вычитаны вновь, на этот случай можно синхронно коммитить после того, как сообщения были обработаны. Хранятся в специальном топике `_consumer_offsets`, который находится у координатора группы (и реплицируется на остальные брокеры). Для разных группы потребителей и одного топика координатор может быть разным.

Пример
```yml
version: '2.4'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.4.4
    container_name: zookeeper-test
    healthcheck:
      test: "[[ $$(echo srvr | nc localhost 2181 | grep -oG 'Mode: standalone') = \"Mode: standalone\" ]]"
      interval: 10s
      timeout: 1s
      retries: 30
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 2181:2181
  
  kafka:
    image: confluentinc/cp-kafka:7.4.4
    container_name: kafka-test
    depends_on:
      zookeeper:
        condition: service_healthy
    healthcheck:
      test: "test $$( /usr/bin/zookeeper-shell zookeper:2181 get /brokers/ids/1 | grep { ) <> ''"
      interval: 3s
      timeout: 2s
      retries: 300
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: "zookeeper:2181"
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092,PLAINTEXT_HOST://localhost:9091
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      KAFKA_JMX_PORT: 9101
      KAFKA_JMX_HOSTNAME: localhost
    ports:
      - "9092:9092"
      - "9091:9091"
      - "9101:9101"
      
  kafdrop:
    image: obsidiandynamics/kafdrop
    container_name: kafdrop-test
    restart: "no"
    ports:
      - "9000:9000"
    environment:
      KAFKA_BROKERCONNECT: "kafka:9092"
    depends_on:
      - "kafka"
```

Открыть kafdrop `http://localhost:9000` и содать топик topic1

Получить список топиков
`docker exec -it kafka-kafka-1 /usr/bin/kafka-topics --list --bootstrap-server localhost:9091`

Отправить сообщение
`docker exec -it kafka-kafka-1 /usr/bin/kafka-console-producer --topic topic1 --bootstrap-server localhost:9091`

Каждая строка - одно сообщение. Прервать Control-C

Открыть в kafdrop топик и посмотреть сообщения в нем (timestamp время отправки сообщения, offset номер сообщения это $2^{64}$)

# Партиции
Топик разделен на партиции (фактически это файл на диске). 
- могут храниться на разных узлах, тем самым повышая скорость чтения и запись в топик (распределенный лог)
- завязаны на механизм доставки сообщения одному "логическому потребителю (consumer)"
- в рамках партиции гарантируется порядок

Можно настроить так чтобы партиции удалялись 
- по достижении определенного размера 
- по времени (хранить до определенной даты)

Consumer group (логический потребитель или группа)
Группа получателей - это один логический потребитель, который представляет из себя многопоточное или распределенное приложение. К каждой партиции привязан только один потребитель из группы.
Но 2 группы могут читать из одного топика.

Пример:
- 3 партиции и 1 потребителя: потребитель читает из каждой партиции
- 3 партиции и 2 потребителя: 1 потребитель читает из 2х партиций, 2 потребитель из 3й партиции
- 3 партиции и 3 потребителя: каждый потребитель читает из своей партиции
- 3 партиции и 4 потребителя: 3 потребителя читают из своей партиции, один простаивает

Отправители и потребители не могут удалять сообщения.

# Ключ
У сообщения в кафка может быть ключ - key. Если есть ключ, то партицирование происходит таким образом, что сообщения с одним ключом попадают в одну партицию

Продьюсер может выбрать партицию в которую отправляет сообщение. По умолчанию он хэш ключа делит на количество партиций и так определяет партицию в которую отправит сообщение

# Перебалансировка
Происходит при добавлении или удалении потребителя в группе, в это время потребители не читают сообщения из топиков

Прочитать сообщения
`docker exec -it kafka-kafka-1 /usr/bin/kafka-console-consumer --from-beginning --topic topic1 --bootstrap-server localhost:9091`

Прочитать сообщения как consumer1
`docker exec -it kafka-kafka-1 /usr/bin/kafka-console-consumer --group consumer1 --topic topic1 --bootstrap-server localhost:9091`

Отправить сообщение с ключом (key:value)
`docker exec -it kafka-kafka-1 /usr/bin/kafka-console-producer --topic topic1 --property "parse.key=true" --property "key.separator=:" --bootstrap-server localhost:9091`

# Репликация
Каждая партиция в топике может иметь несколько реплик (копий), настраивается парамером Replication factor (не может быть больше числа брокеров в кластере)
Обычно когда хотят обеспечить безопасность данных фактор репликации выставляют равным 3 (2 не ставят из-за определенных проблем).

В рамках брокера (экземпляр кафка) может быть до тысячи реплик, относящихся к разным топикам

# Гарантии
- При отправке сообщения продьюсер может сказать сколько in sync replic (те реплики, которые живы) должны подтвердить, чтобы сообщение считалось закоммиченным (0, 1 или all) сообщение помечается commited
- Закоммиченные сообщения не будут потеряны, пока хотя бы 1 реплика жива
- Получатели могут читать только закоммиченные сообщения

# Надежная отправка
При отправке сообщения продьюсер может отправить его с параметром acks
- acks = 0 - если произошла отправка по сети, то она считается успешной
- acks = 1 - отправка считается успешной, если лидер записал это сообщение на диск. Не очень надежно так как если фактор репликации 3, лидер получил сообщение, ответил продьюсеру что все ок, но в этот момент упал, не успев отправить сообщение репликам, тогда из оставшихся реплик выберут лидера в котором нет этого сообщения.
- acks = all - отправка считается успешной, если лидер и все in-sync-replica подтвердили приемку сообщения

Если продьюсер отправил сообщение, брокер принял и сохранил и отправил acks, но его не получил продьюсер, тогда он будет отправлять сообщение повторно. На этот случай в кафке есть режим "идемпотентный продьюсер", тогда дублирования сообщения не будет. Такая гарантия называется "exactly once delivery"

Pull-модель
- чуть хуже latency так как происходит ожидание перед каждым запросом данных
- добавление партиций приводит к ребалансингу
- гарантированный порядок сообщений в рамках партиции

# Кафка vs RabbitMq
Kafka
- высокая пропускная способность
- хранит сообщения даже после чтения, можно перечитать
- репликации из коробки
- сложнее в настройке и запуске
RabbitMq
- лучше latency
- гибче роутинг (exchange/bind/queue)

# Пример отправителя
```scala
import org.apache.kafka.clients.producer.ProducerConfig._
import org.apache.kafka.clients.producer.{KafkaProducer, ProducerRecord}
import org.apache.kafka.common.serialization.StringSerializer

import java.util.Properties

object Prod extends App {
  val props = new Properties()
  props.put(BOOTSTRAP_SERVERS_CONFIG, "localhost:9091")
  props.put(KEY_SERIALIZER_CLASS_CONFIG, classOf[StringSerializer].getName)
  props.put(VALUE_SERIALIZER_CLASS_CONFIG, classOf[StringSerializer].getName)

  val producer = new KafkaProducer[String, String](props)
  val record = new ProducerRecord[String, String]("topic1", "2", "rt")
  producer.send(record) // отправка синхронная, возвращает promise, можно вызвать flush
  producer.close() // блокирующий пока все сообщения не отправим
}
```

# Пример получателя
```scala
import org.apache.kafka.clients.consumer.ConsumerConfig._
import org.apache.kafka.clients.consumer.KafkaConsumer
import org.apache.kafka.common.serialization.StringDeserializer

import java.util.Properties
import scala.concurrent.duration.{FiniteDuration, SECONDS}
import scala.jdk.CollectionConverters.IterableHasAsScala
import scala.jdk.DurationConverters.ScalaDurationOps
import scala.jdk.javaapi.CollectionConverters.asJavaCollection

object Cons extends App {
  val props = new Properties()

  props.put(BOOTSTRAP_SERVERS_CONFIG, "localhost:9091")
  props.put(KEY_DESERIALIZER_CLASS_CONFIG, classOf[StringDeserializer].getName)
  props.put(VALUE_DESERIALIZER_CLASS_CONFIG, classOf[StringDeserializer].getName)
  props.put(GROUP_ID_CONFIG, "default")
  props.put(AUTO_OFFSET_RESET_CONFIG, "earliest") // если хотим читать с начала

  val consumer = new KafkaConsumer[String, String](props)

  consumer.subscribe(asJavaCollection(List("topic1")))

  while (true) {
    val records = consumer.poll(new ScalaDurationOps(FiniteDuration(10, SECONDS)).toJava) // ждем 10с, если нет новых сообщений
    for (record <- records.asScala) {
      println(s"${record.key} ->${record.value}")
    }
  }
}
```