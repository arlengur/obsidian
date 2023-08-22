Независимый фреймворк

подключение в build.sbt
```java
"dev.zio" %% "zio-test" % ZioVersion, 
"dev.zio" %% "zio-test-sbt" % ZioVersion // плагин sbt
	testFrameworks := Seq( new TestFramework("zio.test.sbt.ZTestFramework") 
)
```

Виды тестов
- unit testing 
- property based testing - тесты основанные на свойствах, генерируем набор значений и на них проверяем соблюдение определенных свойств

Пример обычного теста:
```java
object BasicZIOSpec extends DefaultRunnableSpec{ 
	override def spec = suite("Basics")( // пакет тестов
		suite("Arithmetic ops")( // пакет тестов
			test("test case 1"){ // тест
				assert(<expr1>)(<assertion1>)
			}
		)
	) 
}

def test(label: String)(assertion: => TestResult): ZSpec[Any, Nothing] 
def assert[A](expr: => A)(assertion: Assertion[A]): TestResult
```

Тест для zio
```java
suite("Effect testing")( 
	testM("simple effect")( 
		assertM(<expression>)(<assertion>) 
	), 
	... 
)

def testM(label: String)(assertion: => ZIO[R, E, TestResult]): ZSpec[R, E] 
def assertM[R, E, A](effect: ZIO[R, E, A])(assertion:AssertionM[A]): ZIO[R, E, TestResult]
```

Пример:
```java
testM("test with generators"){ 
	check(gen1, gen2){ (genValue1, genValue2) => 
		assert(<property based expr>)(<property based assertion>) 
}
```

Пример обычного теста:
```java
def spec = suite("Basic")(  
	suite("Arithmetic")(  
		test("2*2")(  
			assert(2 * 2)(equalTo(4))  
		),  
		test("division by zero")(  
			assert(2 / 0)(throws(isSubtype[ArithmeticException](anything)))  
		)  
	)
)
```
Пример zio теста:
```java
def spec = suite("Basic")(  
	suite("Effect test")(  
		testM("simple effect")(  
			assertM(ZIO.succeed(2 * 2))(equalTo(4))  
		),  
		testM("test console")(  
			for{  
				_ <- TestConsole.feedLines("Alex", "18")  
				_ <- greeter  
				value <- TestConsole.output  
			} yield{  
				assert(value)(hasSize(equalTo(3))) &&  
				assert(value.head)(equalTo("Как тебя зовут\n"))  
			}  
		),  
		testM("test Clock")(  
			for{  
				fiber <- hello.fork  
				_ <- TestClock.adjust(2 seconds)  
				_ <- fiber.join  
				value <- TestConsole.output  
			} yield assert(value(0))(equalTo("Hello\n"))  
		),  
		testM("counter")(  
			assertM(updateCounterRef)(equalTo(10))  
		)  
	)  
)
```

Аспекты @@nonFlaky - повторяет тест некоторое количество раз
```java
testM("counter")(  
assertM(updateCounterRef)(equalTo(10))  
) @@nonFlaky
```

Stub - просто заглушка, ничего дополнительного не делает
Mock - есть проверяющая логика в моках

Пример:
```java
object EmailServiceMock extends mock.Mock[EmailService]{  
	object SendMail extends Effect[Email, Nothing, Unit]  
	  
	val compose: URLayer[Has[mock.Proxy], EmailService] = ZLayer.fromService{ proxy =>  
		new EmailService.Service {  
			override def sendMail(email: Email): URIO[Console, Unit] =  
				proxy(SendMail, email)  
		  
		}  
	}  
}
```
Effect - когда метод возвращает Zio
Method - когда метод возвращает обычный scala код
Sink, Stream - когда метод возвращает ZioStream

Такой же мок можно получить используя макрос
```java
@mockable[UserDAO.Service]  
object UserDAOMock
```

Пример теста
```java
object UserSpec extends DefaultRunnableSpec{  
	override def spec = suite("User spec"){  
		testM("notify user"){  
			val sendMailMock = EmailServiceMock.SendMail(  
			equalTo(Email(EmailAddress("test@test.com"), Html("Hello here"))), unit  
			)  
			val daoMock = UserDAOMock.FindBy(  
			equalTo(UserID(1)), value(Some(User(UserID(1), EmailAddress("test@test.com"))))  
			)  
			val layer = daoMock >>> UserService.live ++ sendMailMock  
			assertM(UserService.notifyUser(UserID(1)).provideSomeLayer[TestConsole with Console](layer))(anything)  
		}  
	  
	}  
}
```
Schedule

Применение
- сценарии повторения (в случае  ошибок  )
- отложенные вычисления (по расписанию)

Стратегия повторения / восстановления? 
- сколько 
- как часто 
- зависит ли от ошибки

```java
trait Schedule[-Env, -In, +Out]
```
Env - окружение
In - что принимаем на вход
Out - что выдаем на выходе

Категории планировщиков
- повторение 
- задержка 
- точка во времени
```java
object Schedule { 
	def recurs(n: Long): Schedule[Any, Any, Long] 
	def spaced(duration: Duration): Schedule[Any, Any, Long]
	def fixed(interval: Duration): Schedule[Any, Any, Long] 
	def linear(base: Duration): Schedule[Any, Any, Duration]
	def exponential(base: Duration, factor: Double = 2.0): Schedule[Any, Any, Duration]
	def recurWhile[A](f: A => Boolean): Schedule[Any, A, A]
	def recurWhileM[Env, A](f: A => URIO[Env, Boolean]) 
	def recurUntil[A](f: A => Boolean): Schedule[Any, A, A]
	def recurUntilM[Env, A](f: A => URIO[Env, Boolean])
	def secondOfMinute(second: Int): Schedule[Any, Any, Long]
	def minuteOfHour(minute: Int): Schedule[Any, Any, Long]
	def hourOfDay(hour: Int): Schedule[Any, Any, Long] 
	def dayOfWeek(day: Int): Schedule[Any, Any, Long]
	... }
```
- recurs - повторение
- spaced - задержка (время выполнения эффекта не влияет на интервал)
- fixed - задержка (время выполнения эффекта влияет на интервал)
- linear - задержка между повторениями растет линейно
- exponential - задержка между повторениями растет по экспоненте
- recurWhile - повторение, эффект выполняется пока условие истинно
- secondOfMinute - точка во времени

Операторы
```java
trait ZIO[-R, +E, +A] { 
	def repeat[R1 <: R, B](schedule: Schedule[R1, A, B]): ZIO[R1, E, B] 
	def retry[R1 <: R, S](schedule: Schedule[R1, E, S]): ZIO[R1, E, A] 
}
```
- repeat - если zio эффект завершился успешно, то повторяет
- retry - если zio эффект завершился неудачей, то повторяет

Примеры:
```java
val eff = ZIO.effect(println("hello"))  
  
// Написать эффект, котрый будет выводить в консоль Hello 5 раз  
lazy val schedule1 = Schedule.recurs(5)  
lazy val eff1 = eff.repeat(schedule1)  
  
  
// Написать эффект, который будет выводить в консоль Hello 5 раз, раз в секунду  
lazy val schedule2 = Schedule.fixed(1 second)  
lazy val eff2 = eff.repeat(schedule1 && schedule2)  
  
  
// Написать эффект, который будет генерить произвольное число от 0 до 10, и повторяться пока число не будет равным 0  
lazy val schedule3 = Schedule.recurWhile[Int](_ > 0)  
lazy val random = nextIntBetween(0, 11)  
lazy val eff3 = random.repeat(schedule3)  
  
// Написать планировщик, который будет выполняться каждую пятницу 12 часов дня  
lazy val schedule5 = Schedule.dayOfWeek(5) && Schedule.hourOfDay(12)
```

## Ref
Предоставление безопасного доступа к некому состоянию нескольким потокам
```java
trait Ref[A] { 
	def modify[B](f: A => (B, A)): UIO[B] 
	def get: UIO[A] = modify(a => (a, a)) 
	def set(a: A): UIO[Unit] = modify(_ => ((), a)) 
	def update[B](f: A => A): UIO[Unit] = modify(a => ((), f(a))) 
}
```
Пример:
```java
var counter: Int = 0  
  
val updateCounter: UIO[Int] =  
UIO.foreachPar((1 to 5).toList){_ =>  
ZIO.effectTotal(counter += 1)  
}.as(counter)

testM("counter")(  
	assertM(updateCounter)(equalTo(5))  
) @@nonFlaky
```
Ref - обертка над мутабельным состоянием которая позволяет корректно в условиях многопоточности изменять это состояние
```java
trait Ref[A] {  
	def modify[B](f: A => (B, A)): UIO[B]  
	def get: UIO[A] = modify(a => (a, a))  
	def set(a: A): UIO[Unit] = modify(_ => ((), a))  
	def update[B](f: A => A): UIO[Unit] =  
	modify(a => ((), f(a)))  
}

object Ref{  
	def make[A](a: A): UIO[Ref[A]] = ZIO.effectTotal{  
		new Ref[A] {  
			val atomic = new AtomicReference(a)  
			override def modify[B](f: A => (B, A)): UIO[B] = ZIO.effectTotal{  
				var cond = true  
				var r: B = null.asInstanceOf[B]  
				  
				while (cond){  
					val current = atomic.get  
					val tuple = f(current)  
					r = tuple._1  
					cond = !atomic.compareAndSet(current, tuple._2)  
				}  
				r  
			}  
		}  
	}  
}

lazy val updateCounterRef: ZIO[Any, Nothing, Int] = for{  
	counter <- Ref.make(0)  
	_ <- ZIO.foreach_((1 to 10))(_ => counter.get.flatMap(i => counter.set(i + 1)))  
	res <- counter.get  
} yield res

testM("counter")(  
	assertM(updateCounterRef)(equalTo(10))  
) @@nonFlaky
```

RefM не использует cas
```java
trait RefM[A] { 
	def modify[R, E, B](f: A => ZIO[R, E, (B, A)]): ZIO[R, E, B] 
}
```

- Ref и RefM отлично подходят для функциональной работы с состоянием 
- гарантируют атомарность изменения 
- атомарность не поддерживается при композиции (update ее гарантирует)
- используйте Ref там где этого достаточно