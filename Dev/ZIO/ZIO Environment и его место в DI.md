DI - Это возможность одного объекта (компонента, программы) получить доступ к другим объектам, от которых зависит его выполнение при этом без необходимости их явного поиска и/или конструирования.

Дает меньшую связанность между объектами и повышает способность поддержки и тестирования, облегчает понимание и контроль того что и где от чего зависит и происходит.

- безопасность (compile time)
- контроль (понимание того где и какие зависимости используются)
- способность продвижения зависимостей между слоями приложения
- тестирование (насколько это легко)

Конструктор DI 
```java
trait UserService { 
	def notifyUser(userId: UserId): Future[Unit] 
}
trait DBService { 
	def tx(query: Query): Future[QueryResult] 
} 
trait EmailService { 
	def send(email: Email): Future[Unit] 
}

class EmailServiceImpl(smtpServer: SmtpServer) extends EmailService { 
	def send(email: Email): Future[Unit] = { 
		… 
		smtpServer.send(email, …, …) 
		… 
	} 
}
```

```java
class UserServiceImpl(dbService: DBService, emailService: EmailService) extends UserService { 
	def notifyUser(userId: UserId) = { 
		val query: Query = ??? 
		dbService.tx(query).flatMap { r => 
			val email: Email = ??? 
			emailService.send(email) 
		} 
	} 
}
```

## Фреймворк Guis
```java
class CandidateController @Inject()( 
	val authorizationService: AuthorizationService, 
	val candidateService: CandidateService, 
	val candidateStateService: CandidateStateService, 
	val userService: UserService, 
	val journalService: JournalRecordService, 
	val catalogItemService: CatalogItemService, 
	... 
)

object AppBindings{ 
	bindSingleton[AuthorizationService, AuthorizationServiceImpl] 
	bindSingleton[CandidateService, CandidateServiceImp] 
	bindSingleton[CandidateStateService, CandidateStateServiceImp] 
	bindSingleton[UserService, UserServiceImpl] 
	bindSingleton[JournalRecordService, JournalRecordServiceImpl] 
	bindSingleton[CatalogItemService, CatalogItemServiceImpl] 
	... 
}
```

## Tagless final DI 
```java
trait DBService[F[_]] { 
	def tx(query: Query): F[QueryResult] 
} 
trait EmailService[F[_]] { 
	def send(email: Email): F[Unit] 
}

class UserService[F[_]: DBService: EmailService: Monad]() { 
	val query: Query = ??? 
	val dbService = implicitly[DBService[F]] 
	val emailService = implicitly[EmailService[F]] 
	def notifyUser(userId: UserId) = { 
		Monad[F].flatMap(dbService.tx(query)){ r => 
			val email: Email = ??? 
			emailService.send(email) 
		} 
	} 
}
```

## Reader Monad

```java
case class Reader[F[_], -R, B](run: R => F[B]) 
case class Reader[-R, B](run: R => Future[B])

object EmailServiceImpl extends EmailService { 
	def send(email: Email): Reader[Future, SmtpServer, Unit] = Reader{ smtpServer => 
		… 
		smtpServer.send(email, …, …) 
		… 
	} 
}

object UserServiceImpl extends UserService { 
	def notifyUser(userId: UserId): Reader[Future, (DBService, EmailService), Unit] = Reader{ (dbService, emailService) => 
		val query: Query = ??? 
		dbService.tx(query).flatMap { r => 
			val email: Email = ??? 
			emailService.send(email) 
		} 
	} 
}
```

Стандартные подходы к DI
- зависимости через параметры конструктора 
- tagless final (Implicits) 
- Reader

## ZIO Environment DI

Environment как Reader на стероидах 
```java
trait ZIO[-R, +E, +A] { 
	def provide(r: R): IO[E, A] // IO так как избавились от зависимости
} 
object ZIO { 
	def environment[R]: ZIO[R, Nothing, R] = ??? 
}
type MyEnv = Clock with Console with Random 
def e1: ZIO[Clock with Console with Random, Nothing, Unit] 
def e2: ZIO[MyEnv, Nothing, Unit]
```

- порядок 
- повторение 
- identity

Пример:
```java
lazy val e1: ZIO[Random with Clock with Console, Nothing, Unit] = for{ 
	console <- ZIO.environment[Console].map(_.get)  
	clock <- ZIO.environment[Clock].map(_.get)  
	random <- ZIO.environment[Random].map(_.get)  
	_ <- console.putStrLn("Hello").orDie  
	_ <- clock.sleep(5 seconds)  
	int <- random.nextInt  
	_ <- console.putStrLn(int.toString).orDie  
} yield ()
```

Пример:
```java
trait EmailService{  
	def makeEmail(email: String, body: String): Task[Email]  
	def sendEmail(email: Email): Task[Unit]  
}  
  
trait LoggingService{  
	def log(str: String): Task[Unit]  
}  
  
trait UserService{  
	def getUserBy(id: Int): RIO[LoggingService, User]  
}

lazy val getUser: ZIO[LoggingService with UserService, Throwable, User] =  
ZIO.environment[UserService].flatMap(_.getUserBy(1))  
  
lazy val sendMail = ZIO.environment[EmailService].flatMap(_.sendEmail("test@test.com"))  
  
lazy val combined2: ZIO[EmailService with LoggingService with UserService, Throwable, (User, Unit)] =  getUser zip sendMail

// разрешить зависимости
lazy val services: UserService with EmailService with LoggingService = ???
lazy val e3: IO[Throwable, (User, Unit)] = combined2.provide(services)

// частично разрешить зависимости
def f(userService: UserService): UserService with EmailService with LoggingService = ???
lazy val e4: ZIO[UserService, Throwable, (User, Unit)] = combined2.provideSome[UserService](f)
```