![[streamStructure.png|300]]
Позволяют:
- чтение, обработку и запись потока
- throttling (фильровать поток), buffering, back-pressure, windowing
- комбинировать потоки
- распараллеливать потоки
- работать со временем
- работать с состоянием

Stream[F, A] - pull-based поток элементов A, для получения которых выполняется эффект F
Stream[Pure, A] - ленивый поток невыполняющий эффектов

compile запускает стрим
sink получает результат обработки стрима (ex.: toList or drain)

Конструкторы
```scala
val pureApply: Stream[Pure, Int] = Stream.apply(1,2,3)  
  
val ioApply: Stream[IO, Int] = pureApply.covary[IO]  
  
val list = 1 :: 2 :: 3 :: 4 :: Nil  
val str: Stream[Pure, Int] = Stream.emits(list)

val unfolded: Stream[IO, String] = Stream.unfoldEval(0){ s => 
	val next = s +10  
	if (s>=50) IO.none  
	else IO.println(next.toString).as(Some((next.toString, next)))  
}

val s: Stream[IO, Unit] = Stream.eval(IO.readLine).evalMap(x=> IO.println(s">>$x")).repeatN(3)
```

Обратное преобразование
```scala
val a: Seq[Int] = pureApply.toList  
val aa: IO[List[Int]] = ioApply.compile.toList
```

Пример чтения файла
```scala
object Streams extends IOApp.Simple {  
	def openFile: IO[String] = IO.println("open file").as("file descriptor")  
	def closeFile(desc: String): IO[Unit] = IO.println("close file")  
	def readFile(desc: String): Stream[IO, Byte] =  
	Stream.emits(s"File content".map(_.toByte).toArray)  
	  
	val fileResource = Resource.make(openFile)(closeFile)  
	val resourceStream = Stream.resource(fileResource).flatMap(readFile).map(b=> b.toInt + 100)  
	  
	def run: IO[Unit] = {  
		resourceStream.evalMap(IO.println).compile.drain  
	}  
}
```

Обработку потока можно распараллелить
```scala
Stream(1 to 100: _*).chunkN(10).map(println).compile.drain
```

Пример
```scala
def writeToSocket[F[_] : Async](chunk: Chunk[String]): F[Unit] =  
	Async[F].async_{ callback =>  
		println(s"[thread: ${Thread.currentThread().getName}] :: Writing $chunk to socket")  
		callback(Right())  
}  
  
  
  
def run: IO[Unit] = {  
	Stream((1 to 100).map(_.toString) : _*)  
		.chunkN(10)  
		.covary[IO]  
		.parEvalMapUnordered(10)(writeToSocket[IO])  
		.compile  
		.drain
}
```

Планировщик
```scala
// спит между очередным элементом 1 с
val fixedDelayStream = Stream.fixedDelay[IO](1.second).evalMap(_=> IO.println(Instant.now))  
fixedDelayStream.compile.drain

// вытягивает очередной элемент раз в 1 с
val fixedRateStream = Stream.fixedRate[IO](1.second).evalMap(_ => IO.println(Instant.now))
fixedRateStream.compile.drain
```

Создание стрима из очереди
```scala
val queueIO = cats.effect.std.Queue.bounded[IO, Int](100)  
def putInQueue(queue: Queue[IO, Int], value: Int) =  
queue.offer(value)  
  
val queueStreamIO = for {  
	q <- queueIO  
	_ <- (IO.sleep(500.millis) *> putInQueue(q, 5)).replicateA(10).start  
} yield Stream.fromQueueUnterminated(q)  
  
val queueStream = Stream.force(queueStreamIO)

def run: IO[Unit] = {  
	queueStream.evalMap(IO.println).compile.drain
}
```

Преобразование стрима
```scala
def increment(s: Stream[IO, Int]): Stream[IO, Int] = s.map(_+1)

def run: IO[Unit] = {  
	queueStream.through(increment).evalMap(IO.println).compile.drain
}
```

Конкатенация стримов
```scala
queueStream ++ queueStream
```

Параллельная обработка стрима
```scala
resourceStream.parEvalMap(3)(b=> IO.sleep(500.millis) *> IO.println(b)).compile.drain
```