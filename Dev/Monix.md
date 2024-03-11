
Classic Future
- запускается сразу после объявления
- нет встроенного средства отменить вычисления
- выполняется асинхронно то есть всегда на другом потоке
- множественные вложения могут привести к stackoverflow

Monix vs Akka streams
- простое АПИ
- легкий (нет акторного фреймворка)
- лучший контроль выполнения
- легче разобраться в коде
- быстрее
# Алиасы
- `type UIO[A] = IO[Nothing, A]` - эффект не падает и возвращает результат А
- `type Task[A] = IO[Throwable, A]` - эффект падает с ошибкой `Throwable` или возвращает результат А
# Task
- ленивый (ссылочно прозрачный)
- отменяемый
- по умолчанию запускается на том же потоке
- stack and heap safe

```scala
val hello = Task("Hello ")  
val world = Task.evalAsync("World!")  
  
val sayHello = hello  
.flatMap(h => world.map(w => h + w))  
.map(println)  
  
import monix.execution.Scheduler.Implicits.global  
sayHello.runToFuture
```

# Обработка ошибок
```scala
import scala.concurrent.duration._  
def retryOnFailure[A](times: Int, source: Task[A]): Task[A] =  
	source.onErrorHandleWith { err =>  
		// No more retries left? Re-throw error:  
		if (times <= 0) Task.raiseError(err) else {  
		// Recursive call, yes we can!  
		retryOnFailure(times - 1, source)  
			// Adding 500 ms delay for good measure  
			.delayExecution(500.millis)  
	}  
}

retryOnFailure(2, Task(throw new Error("oop")))  
	.onErrorHandle(e => println(e.getMessage))  
	.runToFuture  
Thread.sleep(2000)
```

# Мемоизация
```scala
val rnd = Coeval(Random.nextInt()).memoize  
rnd.value()
rnd.value()
```

```scala
Task(println("boo")).memoize // один раз вычислит Task и потом будет использовать его результат, а обычно каждый раз вычисляет

val p = Task.eval {  
	if (scala.util.Random.nextDouble() > 0.33)  
		throw new RuntimeException("error!")  
	println("moo")  
}.memoizeOnSuccess // запоминает при успешном выполнении

p.onErrorHandle(e => println(e.getMessage)).runToFuture
```

```scala
def sampleMonixTask(a:Int, b:Int): Task[Int] = Task { 
	val result = { 
		a + b 
	} 
	result 
} 
val task = sampleMonixTask(5, 5)
```

# sequence & parSequence
- Task.sequence выполняет задачки последовательно
- Task.parSequence выполняет задачки параллельно

```scala
// Some array of tasks, you come up with something good :-)  
val list: Seq[Task[Int]] = Seq.tabulate(12)(i =>  
	Task {  
		println(s"running func($i)")  
		Thread.sleep(1000)  
		i * 100  
	})  
  
// Split our list in chunks of 30 items per chunk,  
// this being the maximum parallelism allowed  
val chunks = list.sliding(3, 3).toSeq  
  
// Specify that each batch should process stuff in parallel  
val batchedTasks = chunks.map(chunk => Task.parSequence(chunk))  
println(chunks.size)  
// Sequence the batches  
val allBatches = Task.sequence(batchedTasks)  
  
// Flatten the result, within the context of Task  
val all = allBatches.map(_.flatten)  
  
import monix.execution.Scheduler.Implicits.global  
all.runToFuture  
Thread.sleep(5000)
```

# Observable

Создает поток элементов с определенным интервалом, этот поток ленивый, поэтому чтобы он начал работать, на него надо подписаться (subscribe)

Свойства
- ленивый (ссылочно-прозрачный)
- отменяемый
- безопасный (не выполняет небезопасных или блокирующих операций)
- модель один производитель и много потребителей
- неблокирующий back-pressure

```scala
import monix.execution.Scheduler.Implicits.global  
val source = Observable.interval(1.second)  
	// Filtering out odd numbers, making it emit every 2 seconds  
	.filter(_ % 2 == 0)  
	// We then make it emit the same element twice  
	.flatMap(x => Observable(x, x))  
	// This stream would be infinite, so we limit it to 10 items  
	.take(10)  
  
// Observables are lazy, nothing happens until you subscribe...  
val cancelable = source  
	// On consuming it, we want to dump the contents to stdout  
	// for debugging purposes  
	.dump("~")  
	// Finally, start consuming it  
	.subscribe()  
  
Thread.sleep(5000)
```

## Sheduler
контекст на котором выполняются вычисления, является execution context
- отложенные вычисления
- периодические
- позволяет сделать task отменяемым
- использует разные модели выполнения (execution models)

Пример:
```java
Sheduler.computation(name = "computation") // fork join
Sheduler.io(name = "io") // newCachedThreadPool
Sheduler.fixedPool(name = "fixed", 5) 
Sheduler.singleThread(name = "single") 
```

Execution model
- AlwaysAsyncExecution - таск всегда выполняется на другом потоке
- SynchronousExecution - таск выполняется на том же потоке, если нет отдельной конфигурации таск
- BastchExecution - потоки запускаются пачками (нет переключения контекста и потоки не блокируются) по умолчанию

Пример
```java
Task.now(42) // like future
Task.eval(println(42)) // lazy, exec on the same thread
Task(42) // like Task.eval(42).executeAsync
```

Пример переключения контекстов
```java
lazy val io = Scheduler.io("io")  
val source = Task.eval(println(s"Thread: ${Thread.currentThread.getName}"))  
  
import monix.execution.Scheduler.Implicits.global  
source // main thread
.flatMap(_ => source.executeAsync) // global thread
.flatMap(_ => source.executeOn(io)) // io thread
.flatMap(_ => source.asyncBoundary) // global thread
.runSyncUnsafe()
```

Таски можно композировать последовательно
```java
val extract: Task[Seq[String]] = ???
val transform: Seq[String] -> Task[Seq[WTF]] = ???
val load: Seq[WTF] -> Task[Unit] = ???

for {
	strings <- extract
	transformed <- transform(transform)
	_ <- load(transformed)
} yield()
```

и параллельно
```java
val extract1: Task[Seq[String]] = ???
val extract2: Task[Seq[String]] = ???
val extract3: Task[Seq[String]] = ???

val extract = Task.parMap3(extract1, extract2, extract3)(_:+_:+_)
```
чтобы из списка задач получить список результатов
```scala
val tasks: Seq[Task[A]] = Seq(task1, task2, ...)

Task.sequence(tasks) // последовательное выполнение
Task.gather(tasks) // параллельное выполнение с сохранением порядка
Task.gatherUnordered(tasks) // параллельное выполнение с без сохранения порядка
Task.raceMany(tatsks) // получаем результат первый пришедший, все остальные отменяем
```

Гонка потоков
```scala
Task.raceMany
```

Отмена task
```scala
val task = ???
val f: CancelableFuture[Unit] = t.runAsync
f.cancel
```

По умолчанию так является неотменяемым, то есть даже если отмена будет вызвана он завершит свое выполнение
```scala
Task {
	Thread.sleep(100)
	println(42)
}
.doOnCancel(Task.eval(println("On cancel")))
.runAsync
.cancel

Thread.sleep(1000)
```

Но чтобы отменить task нужно вызвать cancelable (отмена происходит на flatMap или на синхронной границе)
```scala
val sleep = Task(Thread.sleep(100)).cancelable
val t = sleep.flatMap(_ => Task.eval(println(42)))
t.runAsync.cancel
Thread.sleep(1000)
```

## Coeval
- Для синхронных или немедленных вычислений
- можно использовать вместо lazy или call-by-name параметров
- не вызывает немедленные вычисления, только при вызове value или run
- позволяет обрабатывать ошибки сайд-эффектов

```scala
import monix.eval.Coeval

val list = List(1, 2)  
val coeval: Coeval[Int] = Coeval(list.sum)  
val value: Int = coeval.value()
```

```scala
import scala.util.{Failure, Success}  
import monix.eval.Coeval

val ex = Coeval { throw new Exception("Error")}  
  
ex.runTry() match {  
	case Success(value) => println(s"Got values $value")  
	case Failure(ex) => println(s"Got error $ex")  
}
```

Как аналог try

```scala
val list = List(1, 2)  
val coeval: Coeval[Int] = Coeval(list.sum)  
val value: Coeval.Eager[Int] = coeval.run()  
  
val ex = Coeval.raiseError[Int](new Exception("Error"))  
val exValue: Coeval.Eager[Int] = ex.run()
```

Преобразовать в Task

```scala
import monix.execution.Scheduler.Implicits.global  
  
val list = List(1, 2)  
val coeval: Coeval[Int] = Coeval(list.sum)  
val task: Task[Int] = coeval.to[Task]  
task.runToFuture
```

Может заменять tailrec

```scala
@tailrec  
def fib(cycles: Int, a: BigInt = 0, b: BigInt = 1): BigInt = {  
	if (cycles < 0) {  
		b  
	} else {  
		fib(cycles - 1, b, a + b)  
	}  
}  
  
def fibM(cycles: Int, a: BigInt = 0, b: BigInt = 1): Coeval[BigInt] = {  
	if (cycles < 0) {  
		Coeval.Now(b)  
	} else {  
		Coeval.defer(Coeval(fib(cycles - 1, b, a + b)))  
	}  
}  
  
fib(4)  
fibM(4)
```

Builders
- now - немедленно вычисляет значение
- eval - аналог lazy, вычисляет в момент вызова
- evalOnce - вычисляет в момент вызова, но только 1 раз, потом возвращает вычисленное значение
- defer - объединяет несколько Coeval 
- raiseError - создание исключения
- unit - создание Unit аналог Coeval.Now(())

restartUntil

```scala
val rnd = Coeval(Random.nextInt()).restartUntil(_ % 2 == 0)  
rnd.value()
```

Освобождение ресурсов

```scala
val rnd = Coeval(1).doOnFinish {  
	case Some(ex) =>  
		println(s"Failure $ex")  
		Coeval.unit  
	case None =>  
		println("Success")  
		Coeval.unit  
}  
rnd.value
```

## Semaphore
```scala
import cats.effect.{ContextShift, IO}  
import cats.implicits._  
import monix.catnap.Semaphore  
import monix.execution.Scheduler

private def makeRequest(i: Int): IO[Unit] = IO(println(s"task $i")) *> IO(Thread.sleep(1000))  
  
implicit val cs: ContextShift[IO] = IO.contextShift(Scheduler.global)  
  
(for {  
	semaphore <- Semaphore[IO](provisioned = 2)  
	tasks = for (i <- 0 until 1000) yield {  
	semaphore.withPermit(makeRequest(i))  
	}  
	// Execute in parallel; note that due to the `semaphore`  
	// no more than 2 tasks will be allowed to execute in parallel  
	_ <- tasks.toList.parSequence  
} yield ()).unsafeRunSync()
```