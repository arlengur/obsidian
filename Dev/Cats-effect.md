# Cats-effect
-  Тайп-классы (cats-effect-kernel)
- Обобщенные структуры данных (cats-effect-std)
- Реализация эффекта (cats-effect)

> [!question] Monad
>[[Monad|Что такое монада]]


# Иерархия монад

![[cats-type-classes-hierarhy.png|400]]

## Короткое замыкание 
Происходит в for-comprehension при первой ошибке цепочка вычислений прерывается
Пример:
```scala
val optionF1 = for {  
	a <- Right(1)  
	b <- Right(1)  
	c <- Left("error")  
	d <- Right(1)  
} yield a + b + c + d  
  
println(optionF1)
```
## IO
IO - тип данных, производящий не ссылочно-прозрачные вычисления
- Ошибка имеет тип Throwable
- Нет Env
`IO[A] ~ ZIO[Any, Throwable, A] ~ zio.Task[A]`

Future сразу запускает вычисление, по завершении которого мы получим значение, при повторном обращении будем получать это значение. `(A => Unit) => Unit`

IO возвращает контейнер с функцией вычисление которой еще не запускалось, оно будет запущено, когда мы попытаемся получить это значение, при повторном обращении будем выполнять вычисление заново. `() => (A => Unit) => Unit`

IO stack safe
```scala
def fib(n: Int, a: Long = 0, b: Long = 1): IO[Long] =
  IO(a + b).flatMap { b2 =>
    if (n > 0) 
      fib(n - 1, b, b2)
    else 
      IO.pure(a)
  }
```
Относительно flatMap используется trampolining

```scala
def apply[A](thunk: => A): IO[A] // alias for delay
def delay[A](thunk: => A): IO[A]

def blocking[A](thunk: => A): IO[A] // uncancelable
def interruptible[A](thunk: => A): IO[A] // cancelable
def interruptibleMany[A](thunk: => A): IO[A] // cancelable
```

Конструкторы
```scala
val pure = IO.pure(println("Hello pure")) // сразу выполняется  
val sideEff = IO.delay(println("Hello side effect")) // отложенное выполнение  
sideEff.unsafeRunSync()  
  
val fromEither = IO.fromEither(Left(new Exception("bom")))  
  
implicit val contextShift = IO.contextShift(ExecutionContext.global)  
val fromFuture = IO.fromFuture(IO.delay(Future.successful(1)))  
  
val failing = IO.raiseError(new Exception("error"))  
val never = IO.never // бесконечный цикл (используется для сервисов)
```
Как встроить нефункциональныф код в функциональный
```scala
implicit val ec = scala.concurrent.ExecutionContext.Implicits.global  
val future = Future(Thread.sleep(2000)).map(_ => 100)  
  
val async = IO.async(  
(cb: Either[Throwable, Int] => Unit) =>  
future.onComplete(a => cb(a.toEither)) // future.onComplete колбэк
)  
async.unsafeRunSync()
println(async.map(_ + 200).unsafeRunSync())  
async.flatMap(i => IO(println(i))).unsafeRunSync()
```

Эффект - это контекст в котором может быть значение. Может быть побочными и непобочными

Пример: 
- Option[A] - F = Option
- Either[MyErr, A] - F = Either[MyErr, ?]
## Монада State
```scala
val stateF = for {  
	a <- State{ (s: Int) => (s+1,s)}  
	b <- State{ (s: Int) => (s+1,s)}  
	c <- State{ (s: Int) => (s+1,s)}  
	d <- State{ (s: Int) => (s+1,s)}  
} yield s"$a $b $c $d"  
println(stateF.runA(10).value)
```

## MonadError
Функциональный аналог try-catch
- наследует возможности Monad
- позволяет прервать вычисление, передавая значение в канал ошибки
- Инстансы
	- MonadError[Option, Unit]
	- MonadError[Either[A, *], A]
	- IO

Пример успешного случая
```scala
type MyMonadError[F[_]] = MonadError[F, String] // фиксация второго параметра  
  
def withErrorHandling[F[_]: MyMonadError]: F[Int] = for {  
	a <- MonadError[F, String].pure(10)  
	b <- MonadError[F, String].pure(10)  
	c <- MonadError[F, String].pure(10)  
	d <- MonadError[F, String].pure(10)  
} yield a + b + c + d 
  
type StringError[A] = Either[String, A]  
  
println(withErrorHandling[StringError]) // Right(40)
```

Пример неуспешного случая
```scala
def withErrorHandling[F[_]: MyMonadError]: F[Int] = for {  
	a <- MonadError[F, String].pure(10)  
	b <- MonadError[F, String].pure(10)  
	c <- MonadError[F, String].raiseError[Int]("fail").ha
	d <- MonadError[F, String].pure(10)  
} yield a + b + c + d 

println(withErrorHandling) // Left(fail)
```
Можно перехватить ошибку способ 1 (выполнение будет остановлено)
```scala
println(withErrorhandling.handleError(_ => 108)) // Right(108)
```
Можно перехватить ошибку способ 2 (выполнение не будет остановлено)
```scala
def withErrorHandling[F[_]: MyMonadError]: F[Int] = for {  
	a <- MonadError[F, String].pure(10)  
	b <- MonadError[F, String].pure(10)  
	c <- MonadError[F, String].raiseError[Int]("fail").handleError(_ => 20)
	d <- MonadError[F, String].pure(10)  
} yield a + b + c + d 

println(withErrorHandling) // Right(50)
```
Можно перехватить ошибку способ 3
```scala
IO.raiseError(new Exception("sdf")).attempt // Left(java.lang.Exception: sdf)
```

MonadCancel
- наследует возможности MonadError
- позволяет прервать вычисление, передавая значение в канал ошибки
- Основные методы: forceR (!>), uncancelable
- Инстансы
	- IO

Пример forceR (при возникновении ошибки продолжает вычисления)
```scala
val b = IO.println(42) *> IO.raiseError(new Exception("sdf")) !> IO.println(10)  
b.unsafeRunSync() // 42 10
```

Пример uncancelable (при возникновении ошибки вычислит значение justSleep)
```scala
val justSleep = IO.sleep(1.second) *> IO.println("not cancelled")  
val justSleepAnfdThrow = IO.sleep(100.millis) *> IO.raiseError(new Exception("error"))  
(justSleep.uncancelable, justSleepAnfdThrow).parTupled.unsafeRunSync()
```

Функциональные эффекты Clock, Random, Unique
Пример
```scala
println(Clock[IO].monotonic.unsafeRunSync)
println(Random.scalaUtilRandom[IO].unsafeRunSync.nextInt.unsafeRunSync)
println(Unique[IO].unique.unsafeRunSync)
```

## Spawn
Предоставляет доступ к зеленым потокам Fiber
- Основные методы: 
	- start - запускает IO  на новом файбере
	- join - ждем завершения файбера
	- cancel - отменяет вычисления на файбере
	- never - запускает бесконечное выполнение файбера
- любой IO выполняется на файбере
- неблокирующий sleep (семантическая блокировка то есть убирает файбер из системного потока, когда поток просыпается планировщик возвращает его для выполнения)

Пример
```scala
object SpawnApp extends IOApp.Simple {  
	def longRunningI(): IO[Unit] =  
		(IO.sleep(500.millis) *>  
			IO.println(s"Thread ${Thread.currentThread}")).iterateWhile(_ => true)  
	  
	def run: IO[Unit] = for {  
		fiber1 <- Spawn[IO].start(longRunningI)  
		_ <- Spawn[IO].start(longRunningI)  
		_ <- IO.println("the fibers has been started")  
		_ <- IO.sleep(2.second)  
		_ <- fiber1.cancel  
		_ <- IO.sleep(1.second)  
	} yield ()  
}
```

## Resource
Resource - гарантирует правильное получение и освобождение ресурсов при любых обстоятельствах
- Основные методы: 
	- make - получение и освобождение ресурсов
	- use - использование ресурсов
## Ref
Ref - изменяемая ячейка памяти с атомарным чтением и записью
Deferred - функциональный аналог Promise
## Reader
A =>  B
- Функция одного аргумента
- Монада
- Может делать композицию функций flatMap
# Tagless Final

Пример без TG
```scala
object FilesAndHttpIO extends IOApp.Simple {  
	def readFile(file: String): IO[String] =  
		IO.pure("scontent of some file")  
	  
	def httpPost(url: String, body: String): IO[Unit] =  
		IO.delay(println(s"Post '$url': '$body'"))  
	  
	def run: IO[Unit] = for {  
		_ <- IO.delay(println("enter file path"))  
		path <- IO.readLine  
		data <-readFile(path)  
		_ <- httpPost("http://mustermann.de", data)  
	} yield ()  
}
```
Пример TG
```scala
trait FileSystem[F[_]] {  
	def readFile(path: String): F[String]  
}  
  
object FileSystem {  
	def apply[F[_]: FileSystem]: FileSystem[F] = implicitly  
}  
  
trait HttpClient[F[_]] {  
	def postData(irl: String, body: String): F[Unit]  
}  
  
object HttpClient{  
	def apply[F[_]: HttpClient]: HttpClient[F] = implicitly  
}  
  
  
trait Console[F[_]] {  
	def readline: F[String]  
	def printLine(s: String) : F[Unit]  
}  
  
object Console{  
	def apply[F[_]: Console]: Console[F] = implicitly  
}  
  
//1. now add the interpreter  
object Interpreter{  
	implicit val consoleIO: Console[IO] = new Console[IO] {  
		def readline: IO[String] = IO.readLine  
		def printLine(s: String): IO[Unit] = IO.println(s)  
	}  
	  
	implicit val fileSystemIO: FileSystem[IO] = new FileSystem[IO] {  
		def readFile(path: String): IO[String] = IO.pure(s"some file with some content $path")  
	}  
	  
	implicit val httpClientIO: HttpClient[IO] = new HttpClient[IO] {  
		def postData(url: String, body: String): IO[Unit] = IO.delay(println(s"POST '$url': '$body'"))  
	}  
}  
  
// bring it together  
object FilesAndHttpTF extends IOApp.Simple{  
	def program[F[_]: Console: Monad: FileSystem :HttpClient]:F[Unit] =  
		for {  
			_ <- Console[F].printLine("Enter file path: ")  
			path <- Console[F].readline  
			data <- FileSystem[F].readFile(path)  
			_<- HttpClient[F].postData("sdfsdfs.de", data)  
		}yield()  
	  
	import Interpreter.consoleIO  
	import Interpreter.fileSystemIO  
	import Interpreter.httpClientIO  
	def run: IO[Unit] = program[IO]
}
```

## Принцип наименьшей силы
Использовать только то что необходимо
```scala
def getUserName[F[_]: Database: Http: Monad](id: UserId): F[String] = ???
```

# Монады трансформеры
Нужны для композирования однородных эффектов

## OptionT
`OptionT[F, A] ~ F[Option[A]]`
Добавляет возможность завершения с отсутствующим значением
```scala
def getUserName: IO[Option[String]] = IO.pure(Some("username"))  
def getId(name: String): IO[Option[Int]] = IO.pure(Some(42))  
def getPermissions(i: Int): IO[Option[String]] = IO.pure(Some("permissions"))

def main(args: Array[String]): Unit = {  
	val res: OptionT[IO, (String, Int, String)] = for {  
	username <- OptionT(getUserName)  
	id <- OptionT(getId(username))  
	permissions <- OptionT(getPermissions(id))  
	} yield (username, id, permissions)  
	  
	println(res.value.unsafeRunSync)
}
```

Значения можно поднимать в контекст
```scala
def getUserName: IO[Option[String]] = IO.pure(Some("username"))  
def getId(name: String): IO[Int] = IO.pure(42)  
def getPermissions(i: Int): IO[Option[String]] = IO.pure(Some("permissions"))  
  
val res: OptionT[IO, (String, Int, String)] = for {  
	username <- OptionT(getUserName)  
	id <- OptionT.liftF(getId(username))  
	permissions <- OptionT(getPermissions(id))  
} yield (username, id, permissions)  
  
println(res.value.unsafeRunSync)
```

## EitherT
`EitherT[F, E, A] ~ F[Either[E, A]]`
Добавляет возможность завершения с ошибкой
```scala
sealed trait UserServiceError  
case class PermissionDenied(msg: String) extends UserServiceError  
case class UserNotFound(userId: Int) extends UserServiceError  
def getUserName(userId: Int): EitherT[IO, UserServiceError, String] = EitherT.pure("user name")  
def getUserAddress(userId: Int): EitherT[IO, UserServiceError, String] =  
EitherT.fromEither(Left(PermissionDenied("Address is not specified")))  
  
def getProfile(id: Int) = for {  
	name <- getUserName(id)  
	address <- getUserAddress(id)  
} yield (name, address)  
  
println(getProfile(2).value.unsafeRunSync())
```
## ReaderT (Kleisli)
`[F, A, B] ~ A => F[B] `
- Композирует функции, которые возвращают монадические значения (является функцией одного аргумента)
- Монада
- Может делать композицию функций flatMap

# Методы
## sequence
последовательно выполняет конкурентный код
```scala
object Main extends IOApp {  
	override def run(args: List[String]): IO[ExitCode] = {  
		val io1 = IO{Thread.sleep(1000); 42}  
		val io2 = IO{Thread.sleep(500); 2}  
		List(io1, io2).sequence.flatMap(r => IO(println(r)))  
	}.as(ExitCode.Success)  
}
```

Можно применять sequence на обычных контейнерах, где для Option 
- будет возвращено  None, если хотя бы один элемент контейнера None,
- либо все элементы контейнера, если среди них нет None
sequence применяется для контейнеров которые являются Applicative, nonEmptySequence применяется для контейнеров которые имеют Apply
```scala
List[Option[String]](Some("hello"), Some("world"), None, Some("again")).sequence // None

List[Option[String]](Some("hello"), Some("world"), Some("again")).sequence // Some(List(hello, world, again))

List[Either[String, Int]](Right(42), Left("err1"), Left("err2"), Right(14)).sequence // Left(err1)

List[Either[String, Int]](Right(42), Right(14)).sequence // Right(List(42, 14))

List[Validated[NonEmptyList[String], Int]](Valid(42), "err1".invalidNel, "err2".invalidNel, Valid(14)).sequence // Invalid(NonEmptyList(err1, err2))

List[Validated[NonEmptyList[String], Int]](Valid(42), Valid(14)).sequence // Valid(List(42, 14))

val nel: NonEmptyList[Map[Int, String]] = NonEmptyList.of(  
	Map(  
		1 -> "one",  
		2 -> "two",  
		3 -> "three"
	),  
	Map(  
		1 -> "one",  
		3 -> "second three",  
		4 -> "four"
	),  
)  
nel.nonEmptySequence // Map(1 -> NonEmptyList(one, one), 3 -> NonEmptyList(three, second three))
```
## traverse
Берет каждый элемент контейнера, применяет на нем функцию и возвращает контейнер/эффект с полученными результатами (последовательно применение map и sequence)
```scala
object Main extends IOApp {  
	override def run(args: List[String]): IO[ExitCode] = {  
		def action(i: Int) = IO.sleep(500.millis).map(_ => i.toString) 
		List.range(1, 10).map(action).sequence.flatMap(r => IO(println(r)))  
		// or
		List.range(1, 10).traverse(action).flatMap(r => IO(println(r)))
		// or
		Option(42).traverse(action).flatMap(r => IO(println(r)))
	}.as(ExitCode.Success)  
}
```

## flattenOption
возвращает все ненулевые элементы контейнера
```scala
List[Option[String]](Some("hello"), Some("world"), None, Some("again")).flattenOption // List(hello, world, again)
```