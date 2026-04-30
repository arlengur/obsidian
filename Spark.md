Делает параллельную обработку данных на кластере в реальном времени.
- Способ параллельного программирования
Поддерживает 
- обработку данных в памяти
- batch или stream обработку

DWH (Data Warehouse, Хранилище данных)
ETL (extract, transform, and load; извлечение, преобразование и загрузка)

Делит большой файл на кусочки (партиции, пр. если ds = 2gb: `2*1024 / 128 = 16 зазделов`), партиции находятся на одном узле кластера, которые обрабатываются на разных серверах в executer'ax (java процесс) - RDD (Resilient Distributed Dataset, Устойчивый распределенный набор данных)

Shuffle — операция перемещения данных по серверам в результате выполнения операций соединения или агрегации Apache Spark.

Driver - управляет executer'ами. Посылает запрос в YARN и говорит сколько нужно executer'ов и с какими мощностями.

Hadoop имеет менеджер ресурсов YARN

Когда запускаем спарк, он обращается к hadoop через yarn и говорит, что нужно сделать.

Halltape — это никнейм специалиста по инженерии данных (Data Engineer)

coalesce(n) - собирает разные партии в n шт разного размера
repatition(n) - собирает разные партии в n шт одинакового размера (делает shuffle и тп)

Основные компоненты:
- Spark Core (Low level Api) - обработка распределенных вычислений. Rdd
- Spark Sql (Structured Api) - обработка данных через Sql. Это абстракция над Rdd
- Structured Streaming - обработка потоковых данных в реальном времени
- MLlib - библиотека для машинного обучения
- GraphX - библиотека для графовых вычислений

Уровни Апи
- RDD работа с распределенными данными на прямую (партиции, кэширование и отказоустойчивость)
- Dataframe - нетипизированное представление данных в виде таблиц (из строк Row и колонок Column), разбитых на разделы. 
- Dataset - типизированное представление данных, разбитых на разделы. 
- Distributed shared variable - переменные типа broadcast и accumulator

SparkContext - управляет подключениями к кластерам и выполняет распределенные вычисления.

Методы spark делятся на трансформации (преобразования объекта) и действия (возвращение результата). Трансформации выполняются только во время действия.

# Пример

https://spark.apache.org/ - Documentation - Latest release - Api Docs - Scala - sql - SparkSession

https://spark.apache.org/ - Documentation - Latest release - Programming guides - Sql - Data Sources - CSV Files

- Создать проект на скала
- добавить зависимости spark-core
- Зайти на https://www.kaggle.com/ - Datasets - найти "stock market dataset"
- В Data Explorer выбрать и скачать csv, создать в проекте папку data и положить его туда 
- В main классе создать сессию

```scala
val spark = SparkSession.builder()  
        .appName("test") // имя приложения
        .master("local[*]") // локальный запуск с количеством экзекуторов соответствующих количеству ядер на машине
        .getOrCreate()
```

- Прочитать файл

```scala
spark.read.csv("data/AAPL.csv")
val df = spark.read  
        .option("header", value = true) // когда есть заголовок
        .csv("data/AAPL.csv")
```

- Запустить

```scala
df.show() // выводит 20 первых строк
```

- Если есть ошибка sparkDriver could not bind on a random free port, то добавить конфигурацию

```scala
val spark = SparkSession.builder()  
        .appName("test")  
        .master("local[*]")  
        .config("spark.driver.bindAddress", "127.0.0.1")  
        .getOrCreate()
```

- Если есть ошибка cannot access class sun.nio.ch.DirectBuffer (тк с Java 17 он не доступен публично), то в Run/Debug Configuration - Modify options - Add VM options и прописать в строку `--add-exports java.base/sun.nio.ch=ALL-UNNAMED`

- Вывести схему таблицы

```scala
val df = spark.read  
        .option("header", value = true)  
        .option("inferSchema", value = true) // определить тип колонок
        .csv("data/AAPL.csv")

df.printSchema()
```

- Вывод колонок

```scala
// 1 способ
df.select("Date", "Open", "Close").show()  
  
// 2 способ
import org.apache.spark.sql.functions.col  
import spark.implicits._  
df.select(col("Date"), $"Open", df("Close")).show()
```

- Преобразования колонок

```scala
val col = df("Open")  
val newCol = col + 2.0  
// val newCol = (col + 2.0).as("inсBy2") переименовали столбец
val strCol = col.cast(StringType)  
val concatCol = concat(strCol, lit(" - str")) // соединение столбцов

// truncate обрезает слишком длинные столбцы
df.select(col, newCol, strColб concatCol).show(truncate = false)

// с фильтрацией
df.select(col, newCol, strCol)
	.filter(newCol > 2.5)
	.filter(newCol === col)
	.show()
```

- Выражения sql

```scala
val tsFromExp = expr("cast(current_timestamp() as string) as tsExp")  
val tsFromFunc = current_timestamp().cast(StringType).as("tsFunc")  
  
df.select(tsFromExp, tsFromFunc).show(truncate = false)

df.selectExpr("cast(Date as string)", "Open + 1.0", "current_timestamp()").show()

// вывод df через sql api 
df.createTempView("df")  
spark.sql("select * from df").show()
```

- Преобразование колонок (переименование, добавление, фильтрация)

```scala
df.withColumnRenamed("Open", "open").show()  

// если несколько колонок
val renamedCols = List(  
    col("Open").as("open"),  
    col("Close").as("close"),  
)  
df.select(renamedCols: _*)  
        .withColumn("diff", col("close") - col("open")) // добавили
        .filter(col("close") > col("open") * 1.1) // отфильтровали
        .show()

// другой вариант
df.select(df.columns.map(c => col(c).as(c.toLowerCase())): _*).show()
```

- Группировка колонок

```scala
// 1 способ
df.select(renamedCols: _*)  
        .groupBy(year(col("date")).as("year"))  
        .agg(  
            functions.max(col("close")).as("maxClose"),  
            functions.avg(col("close")).as("avgClose"))  
        .sort(col("maxClose").desc)  
        .show()

// 2 способ
df.select(renamedCols: _*)  
        .groupBy(year(col("date")).as("year"))  
        .max("close", "high")  
        .show()
```

- Оконные функции (применяются набору строк исходного df по условию partitionBy). В отличие от groupBy не схлопывают строки но добавляют к ним новые колонки

```scala
val window = Window.partitionBy(year(col("date")).as("year"))  
        .orderBy(col("close").desc)  
df.select(renamedCols: _*)  
        .withColumn("rank", row_number().over(window))  
        .filter(col("rank")===1)  
        .sort(col("high").asc)  
        .show()
```

Уровень параллелизма в Spark это количество partition

AST - Abstract Syntax Tree
Catalist - query optimizer

- Чтобы прочитать план который Spark строит по нашим командам нужно применить команду explain(extended = true)
- А если это Rdd, то вызвать toDebugString

- Вывод в файл

```scala
df.write  
        .mode("overwrite")  
        .option("header", value = true)  
        .csv("data/output")
```

# Test

Добавить зависимость https://www.scalatest.org вкладка install
Выбрать стиль вкладка  User Guide - Selecting testing styles

```
"org.scalatest" %% "scalatest" % "3.2.19" % "test",  
"org.scalatest" %% "scalatest-funsuite" % "3.2.19" % "test"
```

Добавить настройку для запуска теста
```
--add-exports
java.base/sun.nio.ch=ALL-UNNAMED
--add-exports
java.base/sun.util.calendar=ALL-UNNAMED
```

Создадим метод
```scala
def highestClosingPricesPerYear(df: DataFrame): DataFrame = {  
    import df.sparkSession.implicits._  
    val window = Window.partitionBy(year($"date")).orderBy($"close".desc)  
    df  
            .withColumn("rank", row_number().over(window))  
            .filter($"rank" === 1)  
            .drop($"rank")  
            .sort($"close".desc)  
}
```

Напишем на него тест
```scala
import org.apache.spark.sql.{Encoder, Encoders, Row, SparkSession}  
import org.apache.spark.sql.types.{DateType, DoubleType, StructField, StructType}  
import org.scalatest.funsuite.AnyFunSuite  
import org.scalatest.matchers.must.Matchers.contain  
import org.scalatest.matchers.should.Matchers.convertToAnyShouldWrapper  
  
import java.sql.Date  
  
class FirstTest extends AnyFunSuite {  
    val spark: SparkSession = SparkSession.builder()  
            .appName("test")  
            .master("local[*]")  
            .getOrCreate()  
  
    private val schema = StructType(Seq(  
        StructField("date", DateType, nullable = true),  
        StructField("open", DoubleType, nullable = true),  
        StructField("close", DoubleType, nullable = true)  
    ))  
  
    test("highestClosingPricesPerYear") {  
        val testRows = Seq(  
            Row(Date.valueOf("2024-01-12"), 1.0, 2.0),  
            Row(Date.valueOf("2025-03-01"), 1.0, 2.0),  
            Row(Date.valueOf("2025-01-12"), 1.0, 3.0),  
        )  
        val expected = Seq(  
            Row(Date.valueOf("2025-01-12"), 1.0, 3.0),  
            Row(Date.valueOf("2024-01-12"), 1.0, 2.0),  
        )  
        implicit val encoder: Encoder[Row] = Encoders.row(schema)  
        val testDf = spark.createDataset(testRows)  
  
        val actualRows = Main.highestClosingPricesPerYear(testDf).collect()  
  
        actualRows should contain theSameElementsAs expected  
    }  
}
```

Пример группировки по 5 объектов в строке
```scala
val schema = StructType(Seq(  
    StructField("id", IntegerType),  
    StructField("publication", StringType)  
))  
val testDF = Seq(  
    Row(1, "pub1"),  
    Row(1, "pub2"),  
    Row(1, "pub3"),  
    Row(1, "pub4"),  
    Row(1, "pub5"),  
    Row(1, "pub6"),  
    Row(1, "pub7"),  
    Row(1, "pub8"),  
    Row(2, "pub9"),  
    Row(2, "pub10"),  
    Row(2, "pub11"),  
    Row(2, "pub12"),  
    Row(2, "pub13"))  
  
implicit val encoder: Encoder[Row] = Encoders.row(schema)  
val df = spark.createDataset(testDF)  
  
  
val window = Window.partitionBy(col("id")).orderBy(col("id").desc)  
  
df.withColumn("rn", row_number().over(window) - 1)  
        .withColumn("bucket", floor(col("rn") / 5))  
        .groupBy("id", "bucket").agg(collect_list("publication").as("publications"))  
        .select("id", "publications")  
        .show(truncate = false)
```