# Введение
SQL базы данных повышают гибкость приложения благодаря транзакционным гарантиям [ACID](Доступ%20к%20БД.md#ACID), а также благодаря своей способности запрашивать данные с помощью JOIN неожиданными способами поверх существующих нормализованных моделей реляционных баз данных.

Учитывая их монолитную/одноузловую архитектуру и использование модели репликации master-slave для избыточности, традиционные SQL базы данных не имеют двух важных особенностей:
- линейной масштабируемости записи (т.е. автоматического разделения на несколько узлов) и 
- автоматической/нулевой потери данных. Это значит, что объем получаемых данных не может превышать максимальную пропускную записи одного узла. Помимо этого, некоторая временная потеря данных должна быть учтена при отказоустойчивости (в архитектуре без разделения ресурсов). Здесь нужно иметь в виду, что недавние коммиты еще не отразились в подчиненной (slave) копии. Обновления без простоя также труднодостижимы в SQL базах данных.

NoSQL базы данных по своей натуре обычно распределенные, т.е. в них данные разбиваются на секции и распределяются по нескольким узлам. Они требуют денормализации. Это означает, что внесенные данные также должны быть скопированы несколько раз для ответа на конкретные запросы, которые вы посылаете. Общая цель состоит в том, чтобы получить высокую производительность путем уменьшения количества шардов, доступных во время чтения. Отсюда следует утверждение, что NoSQL требует от вас моделировать ваши запросы, в то время как SQL требует моделировать ваши данные.

NoSQL акцентируется на достижении высокой производительности в распределенном кластере и это является основным обоснованием множества компромиссов проектирования баз данных, которые включают в себя потерю транзакций ACID, JOIN’ы и согласованные глобальные вторичные индексы.

# PostgreSQL
```
docker run --name my-postgres -p 5432:5432 -e POSTGRES_USER=my_user -e POSTGRES_PASSWORD=123 -e POSTGERS_DB=test -d postgres

docker exec -it my-postgres bash

psql -U my_user -d test

docker stop my-postgres
docker rm my-postgres
```

```sql
CREATE TABLE Music (
    Artist VARCHAR(20) NOT NULL, 
    SongTitle VARCHAR(30) NOT NULL,
    AlbumTitle VARCHAR(25),
    Year INT,
    Price FLOAT,
    Genre VARCHAR(10),
    CriticRating FLOAT,
    Tags TEXT,
    PRIMARY KEY(Artist, SongTitle)
);	
```

```
\d Music
```

```sql
INSERT INTO Music 
    (Artist, SongTitle, AlbumTitle, 
    Year, Price, Genre, CriticRating, 
    Tags)
VALUES(
    'No One You Know', 'Call Me Today', 'Somewhat Famous',
    2015, 2.14, 'Country', 7.8,
    '{"Composers": ["Smith", "Jones", "Davis"],"LengthInSeconds": 214}'
);
INSERT INTO Music 
    (Artist, SongTitle, AlbumTitle, 
    Price, Genre, CriticRating)
VALUES(
    'No One You Know', 'My Dog Spot', 'Hey Now',
    1.98, 'Country', 8.4
);
INSERT INTO Music 
    (Artist, SongTitle, AlbumTitle, 
    Price, Genre)
VALUES(
    'The Acme Band', 'Look Out, World', 'The Buck Starts Here',
    0.99, 'Rock'
);
INSERT INTO Music 
    (Artist, SongTitle, AlbumTitle, 
    Price, Genre, 
    Tags)
VALUES(
    'The Acme Band', 'Still In Love', 'The Buck Starts Here',
    2.47, 'Rock', 
    '{"radioStationsPlaying": ["KHCR", "KBQX", "WTNR", "WJJH"], "tourDates": { "Seattle": "20150625", "Cleveland": "20150630"}, "rotation": Heavy}'
);
```

```sql
SELECT * FROM music;
```

# Cassandra

Создание таблицы в Cassandra очень похоже на PostgreSQL. Одним из основных различий является отсутствие ограничений целостности (например, NOT NULL), но это входит в зону ответственности приложения, а не NoSQL базы данных. 
Первичный ключ состоит из 
- ключа раздела (столбец Artist в примере ниже) и 
- набора столбцов кластеризации (столбец SongTitle в приведенном ниже примере).
Ключ раздела определяет в какой раздел/шард помесить строку, а столбцы кластеризации указывают, как должны быть организованы данные внутри текущего шарда.

```
docker pull cassandra:latest

docker network create cassandra

docker run --rm -d --name cassandra --hostname cassandra --network cassandra cassandra

docker run --rm -it --network cassandra nuvo/docker-cqlsh cqlsh cassandra 9042 --cqlversion='3.4.7'

docker stop cassandra
docker rm cassandra
docker network rm cassandra
```

```yaml
  cassandra:
    image: dcr.qiwi.com/dcr-ext/cassandra:3.11
    container_name: cassandra_native
    volumes:
      - ./cassandra/cassandra_data:/var/lib/cassandra
    ports:
      - 9042:9042
      - 7000:7000
      - 7199:7199
    environment:
      - CASSANDRA_LISTEN_ADDRESS=localhost
      - CASSANDRA_BROADCAST_ADDRESS=host.docker.internal
      - CASSANDRA_USER=cassandra
      - CASSANDRA_PASSWORD=cassandra
      - CASSANDRA_DC=datacenter1
      - CASSANDRA_ENDPOINT_SNITCH=GossipingPropertyFileSnitch
      - TZ=Europe/Moscow
    networks:
      - default
```

```sql
CREATE KEYSPACE IF NOT EXISTS myapp WITH REPLICATION = { 'class' : 'SimpleStrategy', 'replication_factor' : '1' };

USE myapp;
CREATE TABLE Music (
    Artist TEXT, 
    SongTitle TEXT,
    AlbumTitle TEXT,
    Year INT,
    Price FLOAT,
    Genre TEXT,
    CriticRating FLOAT,
    Tags TEXT,
    PRIMARY KEY(Artist, SongTitle)
);

DESCRIBE TABLE MUSIC;
CREATE TABLE myapp.music (
    artist text,
    songtitle text,
    albumtitle text,
    year int,
    price float,
    genre text,
    tags text,
    PRIMARY KEY (artist, songtitle)
) WITH CLUSTERING ORDER BY (songtitle ASC)
    AND default_time_to_live = 0
    AND transactions = {'enabled': 'false'};
```

В целом выражение `INSERT` в Cassandra выглядит очень похоже на аналогичное в PostgreSQL. Однако имеется одно большое различие в семантике. В Cassandra `INSERT` фактически является операцией `UPSERT`, где в строку добавляются последние значения, в случае, если строка уже существует.

# MongoDB

MongoDB организует данные в базы данных (Database) (аналогично Keyspace в Cassandra), где есть коллекции (Collections) (аналогично таблицам), в которых лежат документы (Documents) (аналогично строкам в таблице). В MongoDB в принципе не требуется определение изначальной схемы. Команда «use database», показанная ниже, создает экземпляр базы данных при первом вызове и изменяет контекст для вновь созданной базы данных. Даже коллекции не нужно создавать явно, они создаются автоматически, просто при добавлении первого документа в новую коллекцию. Обратите внимание, что MongoDB по умолчанию использует тестовую базу данных, поэтому любая операция уровня коллекций без указания конкретной базы, будет выполняться в ней по умолчанию.

```shell
docker run -d -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=123 -e MONGO_INITDB_DATABASE=mydb -p 27017:27017 --name my-mongo mongo
docker exec -it my-mongo mongosh

docker stop my-mongo
docker rm my-mongo
```

```yaml
version: '3.0'
services:
  mongo:
   image: mongo
   environment: 
    - MONGO_INITDB_ROOT_USERNAME=arlen
    - MONGO_INITDB_ROOT_PASSWORD=qwerty
    - MONGO_INITDB_DATABASE=mydb
   ports:
      - 27017:27017
   volumes:
      - mongodata:/data/db
volumes:
  mongodata:
   driver: local
```

```sql
use admin
db.auth('arlen','qwerty')

use mydb;
```

В MongoDB insert() похож на PostgreSQL. Добавление данных по умолчанию без `_idspecified` приведет к добавлению нового документа в коллекцию.

```json
db.music.insertOne({  
  artist: "No One You Know",  
  songTitle: "Call Me Today",  
  albumTitle: "Somewhat Famous",  
  year: 2015,  
  price: 2.14,  
  genre: "Country",  
  tags: {  
    Composers: [  
      "Smith",  
      "Jones",  
      "Davis"  
    ],  
    LengthInSeconds: 214  
  }  
});

db.music.insertOne({  
  artist: "No One You Know",  
  songTitle: "My Dog Spot",  
  albumTitle: "Hey Now",  
  price: 1.98,  
  genre: "Country",  
  criticRating: 8.4  
});

db.music.insertOne({  
  artist: "The Acme Band",  
  songTitle: "Look Out, World",  
  albumTitle: "The Buck Starts Here",  
  price: 0.99,  
  genre: "Rock"  
});

db.music.insertOne( {  
  artist: "The Acme Band",  
  songTitle: "Still In Love",  
  albumTitle: "The Buck Starts Here",  
  price: 2.47,  
  genre: "Rock",  
  tags: {  
    radioStationsPlaying: [  
      "KHCR",  
      "KBQX",  
      "WTNR",  
      "WJJH"  
    ],  
    tourDates: {  
      Seattle: "20150625",  
      Cleveland: "20150630"  
    },  
    rotation: "Heavy"  
  }  
});
```

```
show collections; // показать таблицы
```

# Запрос таблицы

Возможно, наиболее существенная разница между SQL и NoSQL с точки зрения составления запросов заключается в использовании формулировок FROM и WHERE. SQL позволяет после выражения FROM выбирать несколько таблиц, а выражение с WHERE может быть какой угодно сложности (включая операции JOIN между таблицами). Однако NoSQL имеет тенденцию накладывать жесткое ограничение на FROM, и работать только с одной указанной таблицей, а в WHERE, всегда должен быть указан первичный ключ. Это связано со стремлением к повышению производительности NoSQL, о котором мы говорили ранее. Это стремление приводит к всяческому уменьшению любого кросс-табличного и кросс-ключевого взаимодействия. Оно может привести к большой задержке в межузловой связи при ответе на запрос и, следовательно, его лучше всего избегать в принципе. Например, Cassandra требует, чтобы запросы были ограничены определенными операторами (разрешены только =, IN, <, >, =>, <=) на ключах разделов, за исключением случаев запроса вторичного индекса (здесь разрешен только оператор =).

Далее будут приведены три примера запросов, которые с лёгкостью могут быть выполнены SQL базой данных.

PostgreSQL

- Вывести все песни исполнителя;
- Вывести все песни исполнителя, совпадающие с первой частью названия;
- Вывести все песни исполнителя, имеющие определенное слово в названии и имеющие цену меньше 1.00.

```sql
SELECT * FROM Music WHERE Artist='No One You Know';

SELECT * FROM Music WHERE Artist='No One You Know' AND SongTitle LIKE 'Call%';

SELECT * FROM Music WHERE Artist='No One You Know' AND SongTitle LIKE '%Today%'
AND Price > 1.00;
```

Cassandra

Из перечисленных выше запросов PostgreSQL только первый будет работать в Cassandra без изменений, поскольку оператор LIKE нельзя применять к столбцам кластеризации, таких как SongTitle. В этом случае допускаются только операторы = и IN.

```sql
SELECT * FROM Music WHERE Artist='No One You Know';
SELECT * FROM Music WHERE Artist='No One You Know' AND SongTitle IN ('Call Me Today', 'My Dog Spot') AND Price > 1.00 ALLOW FILTERING;
```

MongoDB

Основным методом создания запросов в MongoDB является db.collection.find(). Этот метод явно содержит в себе имя коллекции (music в примере ниже), поэтому запрос по нескольким коллекциям запрещен.

```json
db.music.find({
  artist: "No One You Know"
});
db.music.find({
  artist: "No One You Know",
  songTitle: /Call/
});
```

# Считывание всех строк таблицы

Чтение всех строк — это просто частный случай того шаблона запроса, который мы рассматривали ранее.

PostgreSQL & Cassandra
```sql
SELECT * FROM Music;
```

MongoDB
```sql
db.music.find() // вывести все
```

# Редактирование данных в таблице

PostgreSQL

PostgreSQL предоставляет инструкцию UPDATE для изменения данных. Она не имеет возможностей UPSERT, поэтому выполнение этой инструкции завершится ошибкой, в случае если строки больше нет в базе данных.

```sql
UPDATE Music SET Genre = 'Disco' WHERE Artist = 'The Acme Band' AND SongTitle = 'Still In Love';
```

Cassandra

В Cassandra есть UPDATE аналогичный PostgreSQL. UPDATE имеет возможности UPSERT, то есть в случае отсутствия данных выполняет INSERT.

Запрос аналогично примеру в PostgreSQL выше.

MongoDB
Операция updateOne/updateMany. в MongoDB может полностью обновить существующий документ или обновить только определенные поля. По умолчанию она обновляет только один документ с отключенной семантикой UPSERT. Обновление нескольких документов и поведение аналогичное UPSERT можно применить, установив для операции дополнительные флаги. Как например в приведенном ниже примере происходит обновление жанра конкретного исполнителя по его песне.

```sql
db.music.updateMany(
  {"artist": "The Acme Band"},
  { 
    $set: {
      "genre": "New"
    }
  },
  {"upsert": true}
);
```

# Удаление данных из таблицы

PostgreSQL & Cassandra
```sql
DELETE FROM Music WHERE Artist = 'The Acme Band' AND SongTitle = 'Look Out, World';
```

MongoDB

В MongoDB есть два типа операций для удаления документов — deleteOne()/deleteMany() и remove(). Оба типа удаляют документы, но возвращают разные результаты.

```sql
db.music.deleteMany({
        artist: "The Acme Band"
    }
);
```

# Удаление таблицы

PostgreSQL & Cassandra
```sql
DROP TABLE Music;
```

MongoDB
```sql
db.music.drop();
```