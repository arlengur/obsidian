Apache Cassandra is an open source NoSQL distributed database

- Distributed (Распределенная)
	- responsive
	- scalable
- Replicated
	- available
- Masterless
	- No SPoF (Single point of failure)
- Tunable consistency (настраиваемая согласованность)

Масштабируемость
- горизонтальная (когда покупаем такой же сервер и добавляем его к нашему)
- вертикальная (когда более мощный сервер покупаем)

Распределенность данных - на основе поля выбранного как partition key (ключ раздела)

Репликация - когда делаем копии данных на разных серверах.
Replication factor (RF) - количество нод на которые будет размещен 1 раздел сервера. Рекомендуется нечетное число 3, 5 и тд тк тогда CL будет хорошо работать

## Запись
Когда данные приходят, то они сохраняются в Commit log, потом в Memtable и когда происходит flush данные записываются в sstable, которая immutable.

## Чтение
В первую очередь читает из Memtable и во вторую Sstable.
Из Sstable (with bloom filter): сначала читаем из Key Cache, если нет то Partition Summary, если нет то Partiiton Index.

Keyspace - это типа БД
- Table содержат строки и колонки
	- Partitions - раздел содержит группы строк
		- Строка содержит partition key

CQL - Cassandra Query Language

```
CREATE KEYSPACE IF NOT EXISTS cycling 
	WITH REPLICATION = { 
		'class' : 'NetworkTopologyStrategy', // SimpleStrategy for laptop
		'datacenter1' : 3
	};

CREATE TABLE users.cycling_race_winners (
   race_name text, 
   race_position int, 
   PRIMARY KEY ((race_name), race_position)
);
```
users - keyspace
cycling_race_winners - table
race_name - partition key, дб уникальным, все записи с одной race_name попадут в одно место
race_position - clustering column, определяет порядок сортировки, определяют порядок сортировки

Replica node - нода ответственная за данные

Coordinator node - нода получающая данные, каждая нода может выполнять эту роль

CAP theorem (работа может быть сделана быстро, качественно и недорого, но вы можете выбрать 2 пункта)
- Consistency (согласованность - всегда получаем последние записанные данные)
- Availability (высокая доступность - всегда кто-то ответит на запрос данными)
- Partition tolerance (устойчива к разделению сети, какие-то ноды доступны какие-то - нет)

Cassandra - is an AP system (by default)

Consistency Level
- One - координатор нода отправляет запрос на все ноды и после получения ответа от любой из нод возвращает результат
- Two, Three - соответственно от 2х и 3х
- Local_One - ближайшая нода к координатору в ближайшем дата центре (ДЦ)
- Quorum (половина + 1) - координатор нода отправляет запрос на все ноды кластера и после получения ответа от большинства нод возвращает результат
- Local_Quorum (половина + 1) - координатор нода отправляет запрос на все ноды локального ДЦ (ближайшего к нашему приложению) и после получения ответа от большинства нод возвращает результат
- Each_Quorum - координатор нода отправляет запрос на ноды каждого ДЦ, применяется только для write
- ALL -  координатор нода отправляет запрос на все ноды и после получения ответа от всех нод возвращает результат
- ANY - как запрос достиг координатор ноду, то считается обработанным

Immediate Consistency - read CL + write CL > RF
- Рекомендуется read CL = Quorum, write CL = Quorum
- Возможно read CL = One, write CL = Quorum

При настройках по умолчанию acknowledgement от ноды приходит после записи в комит лог

```
Describe keyspaces; // вывести все keyspaces
SELECT * FROM system_schema.keyspaces; // вывести все keyspaces

CREATE KEYSPACE IF NOT EXISTS store WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : '1' };

CREATE TABLE IF NOT EXISTS store.shopping_cart (
userid text PRIMARY KEY,
item_count int,
last_update_timestamp timestamp
);

INSERT INTO store.shopping_cart
(userid, item_count, last_update_timestamp)
VALUES ('9876', 2, toTimeStamp(now()));
INSERT INTO store.shopping_cart
(userid, item_count, last_update_timestamp)
VALUES ('1234', 5, toTimeStamp(now()));


SELECT * FROM killrvideo.users; // не определяет partition key и мы не знаем какую ноду спросить

SELECT firstname FROM killrvideo.users LIMIT 10;
SELECT * FROM killrvideo.users WHERE userid = ... ;
SELECT * FROM killrvideo.users WHERE username = 'ron'; // не даст выполнить
SELECT * FROM killrvideo.users WHERE username = 'ron' ALLOW FILTERING; // даст выполнить, но плохой вариант когда придется опросить все ноды
```

