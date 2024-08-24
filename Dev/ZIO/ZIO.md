История
- 5 июня 2017 года, John A. De Goes впервые говорит о необходимости нового мощного и быстрого типа данных для асинхронного программирования в Scalaz
- 11 июня 2018 года, появляется проект ZIO, как самостоятельная библиотека без зависимостей. 

Суть
- Прямая поддержка конкурентности, с возможностью безопасной отмены запущенных вычислений
- Типизированные ошибки
- Типизированный контекст
- Простота использования и быстродействие

`trait ZIO[-R, +E, +A]` то есть это как функция из окружения `R => Either[E, A]`, где Е - неуспех, А - успех
R - тип окружения (от чего ZIO зависит)
E - тип ошибки, с которым тайп-параметр может завершиться
A - тип результата в случае успешного завершения

Пример:
`val f: String => Either[Throwable, Int] = ???` чтобы получить результат мы должны подставить строку

Ментальная Модель (функциональные эффекты как шаблон)

Пример (используя executable encoding реализуем свой zio) :
```scala
  case class ZIO[-R, +E, +A](run: R => Either[E, A]) {
    self =>
    def map[B](f: A => B): ZIO[R, E, B] = flatMap(a => ZIO(_ => Right(f(a))))
    // или
    def map[B](f: A => B): ZIO[R, E, B] = ZIO(r => self.run(r).map(f))

    def flatMap[R1 <: R, E1 >: E, B](f: A => ZIO[R1, E1, B]): ZIO[R1, E1, B] =
      ZIO(r => self.run(r).fold(
        e => fail(e),
        v => f(v)
    ).run(r))
  }

  object ZIO {
    def effect[A](a: => A): ZIO[Any, Throwable, A] = {
	    try {  
			ZIO(_ => Right(a))  
		} catch {  
			case e: Throwable => ZIO(_ => Left(e))  
		}  
		// или  
		ZIO(_ => Try(a).toEither)
    }

    def fail[E](e: E): ZIO[Any, E, Nothing] = ZIO(_ => Left(e))
  }

// консольное echo приложение с помощью нашего ZIO
   val echo: ZIO[Any, Throwable, Unit] = for{
     str <- ZIO.effect(StdIn.readLine())
     _ <- ZIO.effect(println(str))
   } yield()

// Запуск
echo.run(())
  ```

## Алиасы
```scala
type IO[+E, +A] = ZIO[Any, E, A] // не нужно окружения, можем упасть с ошибкой или выполниться успешно
type Task[+A] = ZIO[Any, Throwable, A] // не нужно окружения, можем упасть с ошибкой или выполниться успешно (аналог Future или cats IO)
type RIO[-R, +A] = ZIO[R, Throwable, A] // нужно окружение, можем упасть с ошибкой или выполниться успешно
type UIO[+A] = ZIO[Any, Nothing, A] // не нужно окружения, не можем упасть с ошибкой (unfailable) и можем выполниться успешно
type URIO[-R, +A] = ZIO[R, Nothing, A] // нужно окружение, не можем упасть с ошибкой (unfailable) и выполниться успешно
```

## Конструкторы
```scala
- ZIO.succeed(): UIO
- ZIO.effect(): Task // ZIO.attempt since 2.0
- ZIO.fromFuture(ec => f): Task
- ZIO.fromTry(): Task
- ZIO.fromOption(): IO[Option[Nothing], Int]
- ZIO.fromOption().option: UIO[Option[Int]]
- ZIO.fromOption().option.some: IO[Option[Nothing], Int]
- ZIO.fromEither(): IO[String, Int]
- ZIO.fromEither().either: UIO[Either[String], Int]
- ZIO.fromEither().absolve: IO[String, Int]
- ZIO.fromFunction[String, Unit](str => println("")): IO[String, Unit]
- ZIO.unit: UIO
- ZIO.none: UIO[Option[Nothing]]
- ZIO.never: UIO[Nothing] // while(true)
- ZIO.die(new Throwable("Died"): UIO[Nothing]
- ZIO.fail("Ooops"): IO[String, Nothing]
// check https://zio.dev/guides/migrate/zio-2.x-migration-guide/
```

Пример ротации типа:
```scala
  type User
  type Address

  def getUser(): Task[Option[User]] = ???
  def getAddress(u: User): Task[Option[Address]] = ???

  val r = for{
    user <- getUser().some
    address <- getAddress(user)
  } yield address
```

## Комбинаторы
```scala
a.zip(b): Task[(Int, String)] // последовательная комбинация эффектов a и b
a.zipRight(b): Task[String] // последовательная комбинация эффектов a и b, игнорируем то что слева (*> алиас)
b.zipLeft(a): Task[String] // последовательная комбинация эффектов a и b, игнорируем то что справа (<* алиас)
b.zipWith(b)(_ + _): Task[String] // Последовательная комбинация эффета b и b, при этом результатом должна быть конкатенация возвращаемых значений
a.orElse(b) // Другой эффект в случае ошибки
a.as(1): Task[Int] // as игнорируем результат A и применяет B 
```

Запуск:
```scala
object Main2 extends zio.App{
  override def run(args: List[String]): URIO[zio.ZEnv, ExitCode] = zioConstructors.z11.exitCode
}

zio.Runtime.default.unsafeRun(multipleErrors.app)
```

## Рекурсия

Пример:
```scala
// ZIO эффект который будет писать строку в консоль
def writeLine(str: String): Task[Unit] = ZIO.attempt(println(str))

// ZIO эффект который будет читать число в виде строки из консоли 
val readLine: Task[String] = ZIO.attempt(StdIn.readLine()) 

// ZIO эффект котрый будет трансформировать читающий строку из консоли в эффект содержащий Int
val readInt: Task[Int] = readLine.flatMap(str => ZIO.attempt(str.toInt))

//  считывает из консоли Int введенный пользователем, а в случае ошибки, сообщает о некорректном вводе, и просит ввести заново
def readIntOrRetry: Task[Int] = readInt.orElse {
    ZIO.attempt(println("Некорректный ввод, попробуйте еще")) zipRight readIntOrRetry
  }
```

Пример:
```scala
// Написать ZIO версию факториала
  def factorialZ(n: BigInt): Task[BigInt] = {
    if( n <= 1) ZIO.succeed(n)
    else ZIO.succeed(n).zipWith(factorialZ(n - 1))(_ * _)
  }
```

# Error vs Defect

Error - это ожидаемая ошибка, фиксируемая на уровне типа эффекта
`val eff: IO[NumberFormatException, Int] = ???`

Defect - непредвиденная ошибка, не знаем, как восстановиться
`val eff: UIO[Int] = ???`

Пример:
```scala
def readFile(fileName: String): ZIO[Any, Nothing, List[String]] // если упадем, то не знаем что делать, поэтому UIO[List[String]]
```

Пример:
```scala
def parseConfig(fileName: String): ZIO[Any, IOException, Config] // если падаем, то можем вернуть дефолтный конфиг, поэтому IO[IOException, Config]
```

Когда падаем с ошибкой, то на самом деле получаем 

Cause - где лежит ошибка, позволяет отследить множественные и конкурентные ошибки

```scala
sealed trait Cause[+E]
object Cause {
final case class Die(t: Throwable) extends Cause[Nothing] // это Defect
final case class Fail[+E](e: E) extends Cause[E] // это Error
}
```

Успешное выполнение на самом деле возвращает Exit (похожа на Either)

```scala
sealed trait Exit[+E, +A]
object Exit{
final case class Success[+A](value: A) extends Exit[Nothing, A]
final case class Failure[+E](cause: Cause[E]) extends Exit[E, Nothing]
```

Теперь наша ментальная модель от
`ZIO[-R, +E, +A] = R => Either[E, A]`

станет
`ZIO[-R, +E, +A] = R => Either[Cause[E], A]`

Пример:
```scala
  case class ZIO[-R, +E, +A](run: R => Either[E, A]) {self =>
    // Базовый оператор для работы с ошибками
    def foldM[R1 <: R, E1, B](
             failure: E => ZIO[R1, E1, B],
             success: A => ZIO[R1, E1, B]
             ): ZIO[R1, E1, B] =
      ZIO(r => self.run(r).fold(
        e => failure(e),
        a => success(a)
      ).run(r))


    def orElse[R1 <: R, E1 >: E, A1 >: A](other: ZIO[R1, E1, A1]): ZIO[R1, E1, A1] =
      foldM(
        _ => other,
        v => ZIO(r => Right(v))
      )

    // метод, который будет игнорировать ошибку в случае падения, а в качестве результата возвращать Option
    def option: ZIO[R, Nothing, Option[A]] =
      foldM(
        _ => ZIO(_ => Right(None)),
        v => ZIO(_ => Right(Some(v)))
      )

    // метод, который будет работать с каналом ошибки
    def mapError[E1](f: E => E1): ZIO[R, E1, A] = foldM(
      e => ZIO(_ => Left(f(e))),
      v => ZIO(_ => Right(v))
    )
  }
```

Пример:
```scala
  sealed trait IntegrationError
  case object ConnectionFailed extends IntegrationError
  case object CommunicationFailed extends IntegrationError

  lazy val connect: IO[ConnectionFailed.type, Unit] = ???
  lazy val getSomeData: IO[CommunicationFailed.type, String] = ???
  lazy val connectAndGetData: ZIO[Any, IntegrationError, String] = connect zipRight getSomeData
```

Пример:
```scala
  lazy val io1: IO[String, String] = ???
  lazy val io2: IO[Int, String] = ???

   val z1: IO[Any, (String, String)] = io1 zip io2
```

Пример:
```scala
// избежать потерю информации об ошибке, в случае композиции?
lazy val io3: IO[Either[String, Int], (String, String)] = io1.mapError(Left(_)).zip(io2.mapError(Right(_)))
```

Пример:
```scala
  def either: Either[String, Int] = ???
  def errorToErrorCode(str: String): Int = ???

  lazy val effFromEither: IO[String, UserId] = ZIO.fromEither(either)

  // Залогировать ошибку effFromEither, не меняя ее тип и тип возвращаемого значения
  lazy val z2: IO[String, Int] = effFromEither.tapError { e =>
    ZIO.effect(println(e)).orDie
  }
```

Пример:
```scala
    type User = String
    type UserId = Int

    sealed trait NotificationError
    case object NotificationByEmailFailed extends NotificationError
    case object NotificationBySMSFailed extends NotificationError

    def getUserById(userId: UserId): Task[User] = ???

    def sendEmail(user: User, msg: String): IO[NotificationByEmailFailed.type, Unit] = ???

    def sendSMS(user: User, msg: String): IO[NotificationBySMSFailed.type, Unit] = ???

    def sendNotification(userId: UserId): IO[NotificationError, Unit] = for{
      user <- getUserById(1).orDie
      _ <- sendEmail(user, "email")
      _ <- sendSMS(user, "9999")
    } yield ()
```

В модели ZIO можно получить доступ к каналу ошибки
```scala
trait ZIO[-R, +E, +A] {
 ...
def sandbox: ZIO[R, Cause[E], A] // доступ к Cause
def unsandbox[E1]: ZIO[R, E1, A] // вернулись обратно к E1
def foldCauseM[R1 <: R, E1, B](
 failure: Cause[E] => ZIO[R1, E1, B],
 success: A => ZIO[R1, E1, B]): ZIO[R1, E1, B] = ???
}
```

# Множественные ошибки
```scala
trait Error extends Product
case object E1 extends Error
case object E2 extends Error

object multipleErrors{
    val z1: IO[E1.type, Int] = ZIO.fail(E1)

    val z2: IO[E2.type, Int] = ZIO.fail(E2)

    val result = z1 zipPar z2

    val app = result.tapCause{// логирование
        case Both(left, right) =>
        ZIO.effect(println(left.failureOption)) zipRight( ZIO.effect(println(right.failureOption)))
    }
}
```
