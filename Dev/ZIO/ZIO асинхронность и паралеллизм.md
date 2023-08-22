Fiber model - концепция зеленых потоков (передача управления потоками из пространства ядра ОС в пространство пользователя, в пространство JVM)

- Легковесный аналог системного потока 
- Моделирует исполняемое вычисление 
- Инструкции выполняются последовательно
- Дешевы в создании 
- Безопасно прерывать 
- Join без блокировки
![[ZioFiberModel.png|500]]

![[ZioFiberProgram.png|500]]

В момент вызова unsafeRun создается Fiber он отправляется в Executor, который занимается орекестрацией файберов и выполнением инструкций из них.

Любой Zio эффект выполняется на файбере.

Операторы
```java
trait ZIO[-R, +E, +A]{ 
	def fork: URIO[R, Fiber[E, A]] 
}
trait Fiber[+E, +A]{ 
	def join: IO[E, A] 
}
```

fork - возвращает Zio с файбером
join - получает результат файбера

Live
```java
// Напишите эффект, который будет считать время выполнения любого эффекта  
def printEffectRunningTime[R, E, A](zio: ZIO[R, E, A]): ZIO[R with Clock, E, A] = for{  
	start <- currentTime  
	r <- zio  
	end <- currentTime  
	_ <- ZIO.effect(println(s"Running time ${end -start}")).orDie  
} yield r  
  
// Эффект который все что делает, это спит заданное кол-во времени, в данном случае 1 секунду  
lazy val sleep1Second = ZIO.sleep(1 seconds)  
  
// Эффект который все что делает, это спит заданное кол-во времени, в данном случае 3 секунды  
lazy val sleep3Seconds = ZIO.sleep(3 seconds)  
  
// Создать эффект который печатает в консоль GetExchangeRatesLocation1 спустя 3 секунды  
lazy val getExchangeRatesLocation1 = sleep3Seconds zipRight  
ZIO.effect(println("GetExchangeRatesLocation1"))  
  
// Создать эффект который печатает в консоль GetExchangeRatesLocation2 спустя 1 секунду  
lazy val getExchangeRatesLocation2 = sleep1Second zipRight  
ZIO.effect(println("GetExchangeRatesLocation2"))  
  
// Написать эффект котрый получит курсы из обеих локаций  
lazy val getFrom2Locations = getExchangeRatesLocation1 zip getExchangeRatesLocation2
```
Запуск последовательно (4с):
```java
zio.Runtime.default.unsafeRun(printEffectRunningTime(getFrom2Locations))
```

Пример 2:
```java
// Написать эффект котрый получит курсы из обеих локаций паралельно  
lazy val getFrom2Locations2 = for{  
	f2 <- getExchangeRatesLocation1.fork  
	r2 <- getExchangeRatesLocation2 
	r1 <- f1.join  
} yield (r1, r2)
```

Запуск параллельно (3с):
```java
zio.Runtime.default.unsafeRun(printEffectRunningTime(getFrom2Locations2))
```

for-comprefension выполняет код последовательно, а выполнение на файберах (когда делаем fork) конкурентно

Пример
```java
// Предположим нам не нужны результаты, мы сохраняем в базу и отправляем почту  
lazy val writeUserToDB = sleep1Second zipRight ZIO.effect(println("writeUserToDB"))  
lazy val sendMail = sleep1Second zipRight ZIO.effect(println("sendMail"))  
  
// Написать эффект котрый сохранит в базу и отправит почту паралельно  
lazy val writeAndSend = for{  
_ <- writeUserToDB.fork  
_ <- sendMail.fork  
} yield ()
```

Запуск параллельно (в консоли будет только залогировано время так как файбер в котором делаем fork является родительским и когда он завершается он завершает все дочерние файберы):
```java
zio.Runtime.default.unsafeRun(printEffectRunningTime(writeAndSend))
```

А если сделать так 
```java
// Предположим нам не нужны результаты, мы сохраняем в базу и отправляем почту  
lazy val writeUserToDB = ZIO.effect(println("writeUserToDB")) zipRight sleep1Second    
lazy val sendMail = ZIO.effect(println("sendMail")) zipRight sleep1Second  
```

В этом случае в консоль будет выведено:
```
writeUserToDB
sendMail
Running time 0
```
Файберы также будут прерваны, но успеют вывести сообщения в консоль.

По умолчанию файберы не демоны, чтобы сделать их демонами надо вызвать forkDaemon.

## Fiber supervision model

![[ZioSupervisionModel.png|500]]
В первом случае fiber0 родитель fiber1, а fiber1 родитель fiber2 
```java
lazy val zio1: Task[Int] = ??? 
lazy val zio2: URIO[Any, Fiber.Runtime[Throwable, Int]] = zio1.fork 
lazy val zio3: URIO[Any, Fiber.Runtime[Nothing, Fiber.Runtime[Throwable, Int]]] = zio2.fork 
```
Во втором случае fiber0 родитель zio2 и fiber1 с zio1
```java
val zio4 : ZIO[Any, Throwable, (Fiber.Runtime[Throwable, Int], String)] = zio1.fork.zip(ZIO.effect("Hi"))
```

- У каждого файбера есть своя область видимости (scope) 
- Каждый файбер форкается в некой области видимости (имеет родителя)
- Каждый файбер форкается в области видимости текущего, если иное не задано явным образом
- Когда файбер останавливается по любой из причин (успех, падение, прерывание), его область видимости закрывается 
- Когда закрывается область видимости файбера, то ВСЕ файберы форкнутые в его области видимости ПРЕРЫВАЮТСЯ

## Прерывание выполнения эффекта

Пример прерывания:
```java
lazy val greeter = (sleep1Second zipRight ZIO.effect(println("Hello"))).forever.fork  
  
lazy val g1 = for{  
f1 <- greeter  
_ <- ZIO.sleep(2 seconds)  
_ <- f1.interrupt  
_ <- ZIO.sleep(3 seconds)  
} yield ()

zio.Runtime.default.unsafeRun(printEffectRunningTime(g1))
```

Пример бесконечного выполнения:
```java
lazy val g1 = for{  
f1 <- ZIO.effect(while (true) println("hello")).fork  
_ <- ZIO.sleep(2 seconds)  
_ <- f1.interrupt  
_ <- ZIO.sleep(3 seconds)  
} yield ()

zio.Runtime.default.unsafeRun(printEffectRunningTime(g1))
```

Параллельные операторы

zipPar параллельно запускает выполнение:
```java
lazy val p1 = getExchangeRatesLocation1 zipPar getExchangeRatesLocation2
```

race параллельно запускает выполнение и возвращает результат первого выполненого потока а второй прерывает:
```java
lazy val p1 = getExchangeRatesLocation1 zipPar getExchangeRatesLocation2
```

Параллельный запуск в разных потоках
```java
lazy val p3 = ZIO.foreachPar(List(1, 2, 3, 4))( el =>  
ZIO.effect(println(el)) zipRight sleep1Second
```

Параллельный запуск в 2 потока
```java
lazy val p3 = ZIO.foreachParN(2)(List(1, 2, 3, 4))( el =>  
ZIO.effect(println(el)) zipRight sleep1Second
```

## Locking 
```java
trait ZIO[-R, +E, +A] { 
	def lock(executor: Executor): ZIO[R, E, A] 
	def on(executionContext: ExecutionContext): ZIO[R, E, A] 
}
```

- Когда эффект залочен на Executor, все части этого эффекта будут выполнены на этом Executor 
- Внутренние скоупы имеют более высокий приоритет

Пример выполнения на одном Executor:
```java
lazy val doSomething: UIO[Unit] = ???  
lazy val doSomethingElse: UIO[Unit] = ???  
  
lazy val executor: Executor = ???  
  
lazy val eff = for{  
f1 <- doSomething.fork  
_ <- doSomethingElse  
r <- f1.join  
} yield r  
  
lazy val result = eff.lock(executor)
```

Пример выполнения на 2х Executor'ax:

```java
lazy val executor1: Executor = ???  
lazy val executor2: Executor = ???  
  
lazy val eff2 = for{  
f1 <- doSomething.lock(executor2).fork  
_ <- doSomethingElse  
r <- f1.join  
} yield r  
  
lazy val result2 = eff2.lock(executor1)
```

doSomething выполнится на executor2, а doSomethingElse на executor1