Future
- запускается сразу после объявления
- нет встроенного средства отменить вычисления
- выполняется асинхронно то есть всегда на другом потоке
- множественные вложения могут привести к stackoverflow

Task
- ленивый (ссылочно прозрачный)
- отменяемый
- по умолчанию запускается на том же потоке
- stack and heap safe

Sheduler
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

Observable[A]
Обработка потока данных

Свойства
- ленивый (ссылочно-прозрачный)
- отменяемый
- безопасный (не выполняет небезопасных или блокирующих операций)
- модель один производитель и много потребителей
- неблокирующий back-pressure

Monix vs Akka streams
- простое АПИ
- легкий (нет акторного фреймворка)
- лучший контроль выполнения
- легче разобраться в коде
- быстрее

Пример https://github.com/ilya-murzinov/seuraajaa