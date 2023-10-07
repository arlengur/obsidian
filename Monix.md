# Алиасы
- `type UIO[A] = IO[Nothing, A]` - эффект не падает и возвращает результат А
- `type Task[A] = IO[Throwable, A]` - эффект падает с ошибкой `Throwable` или возвращает результат А

# Task
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