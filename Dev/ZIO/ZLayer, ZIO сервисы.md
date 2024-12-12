ZLayer решает проблему предоставления комбинации сервисов
```java
trait DBService{ 
	def tx(query: Query): IO[DBError, QueryResult] 
} 
trait EmailService{ 
	def sendEmail(email: Email): Task[Unit] 
}

val ef1: ZIO[DBService, DBError, QueryResult] = ??? 
val ef2: ZIO[EmailService, Throwable, Unit] = ??? 
val ef3: ZIO[EmailService with DBService, Any, QueryResult] = ef1 <* ef2 // <* == zipLeft

val services: EmailService with DBService = ???
zio.Runtime.default.unsafeRun(ef3.provide(services))
```

Проблема в том что реализовать services может быть очень затратно
```java
val services: DBService with EmailService = new DBService with EmailService { 
	override def tx(query: Query): IO[DBError, QueryResult] = dBService.tx(query) 
	override def sendEmail(email: Email): Task[Unit] = emailService.sendEmail(email) 
}
```
## `ZLayer[-RIn, +E, +ROut]`
Функциональный эффект для узкой предметной области. 

- Описывает слой приложения 
- Представляет собой рецепт, как на основании одних сервисов построить другие 
- При создании слоя можно использовать эффекты и ресурсы 
- Слои в декларациях можно использовать несколько раз
- Возможность строить отдельные сервисы 
- Возможность строить комбинации сервисов 
- Возможность частично избавляться от зависимостей

```scala
type RLayer[-RIn, +ROut] = ZLayer[RIn, Throwable, ROut] 
type URLayer[-RIn, +ROut] = ZLayer[RIn, Nothing, ROut] 
type Layer[+E, +ROut] = ZLayer[Any, E, ROut] 
type ULayer[+ROut] = ZLayer[Any, Nothing, ROut] 
type TaskLayer[+ROut] = ZLayer[Any, Throwable, ROut]
```

U - Nothing в канале ошибки
## Has 
Некая тайп-левел мапа которая в качестве ключей хранит тип а в качестве значений экземпляры объектов.

```java
val e = for{ 
	consoleService <- ZIO.environment[Console].map(_.get) // get получает из тайп-левел мапы конкретный объект
	_ <- consoleService.putStrLn("Hello") 
} yield ???
```

```scala
Has[DBService] 
Map[DBServiceType -> DBServiceImpl] 

Has[DBService] with Has[EmailService] 
Map( 
	DBServiceType -> DBServiceImpl, 
	EmailServiceType -> EmailServiceImpl 
)
```
### ZLayer конструкторы 
- succeed - создает заведомо непадающий слой
- fromService - когда при создании слоя он зависит от другого (UserService зависит от EmailService)
- fromServices - когда зависимостей много
- fromFunction - когда зависимости указываем через тайп-алиас
```java
val userLayer: ULayer[Has[UserService]] = 
	ZLayer.succeed(new UserService{ 
		override def notifyUser(userID: UserID): RIO[EmailService with Console, Unit] = ??? 
	})

val userLayer: ZLayer[Has[EmailService], Nothing, Has[UserService]] = 
	ZLayer.fromService[EmailService, UserService](es => new UserService(es){ 
		override def notifyUser(id: UserID): RIO[...] = ??? 
	})

val userLayer: ZLayer[Has[A] with Has[B], Nothing, Has[C]] = ZLayer.fromServices[A, B, C]((a, d) => new C(a, b){ .... } )

type Env = DBService with EmailService 
val userLayer: ZLayer[Env, Nothing, Has[UserService]] = ZLayer.fromFunction[Env, UserService] { env => 
	val dbService = env.get[DBService] 
	val emailService = env.get[EmailService] 
	new UserService(dbService, emailService){ ... } 
}

ZLayer.fromServiceMany(...) 
ZLayer.fromManaged(...) 
ZLayer.fromAcquireRelease(...)
```

Пример:
```java
package object emailService {
// 1 компонент  
type EmailService = Has[EmailService.Service]  
  
// 2 компонент  
object EmailService {  
	trait Service { // интерфейс  
		def sendMail(email: Email): URIO[zio.console.Console, Unit]  
	}  
  
// боевой слой описывающий работу сервиса  
	val live = ZLayer.succeed(new Service {  
		override def sendMail(email: Email): URIO[Console, Unit] = zio.console.putStrLn(email.toString).orDie  
	})
}
}
case class Email(to: EmailAddress, body: Html)  
case class EmailAddress(address: String) extends AnyVal  
case class Html(raw: String) extends AnyVal

package object userDAO {
// 1  
type UserDAO = Has[UserDAO.Service]  
  
//2  
object UserDAO{  
	trait Service{  
		def list(): Task[List[User]]  
		def findBy(id: UserID): Task[Option[User]]  
	}  
	  
	val live: ULayer[UserDAO] = ???   
}
}

package object userService {
type UserService = Has[UserService.Service]  
 
object UserService{  
	trait Service{  
		def notifyUser(id: UserID): RIO[UserDAO with EmailService with Console, Unit]  
	}  

	// описали реализацию notifyUser
	class ServiceImpl extends Service{  
		override def notifyUser(id: UserID): RIO[UserDAO with EmailService with Console, Unit] = for{  
		userDAO <- ZIO.environment[UserDAO].map(_.get)  
		emailService <- ZIO.environment[EmailService].map(_.get)  
		user <- userDAO.findBy(UserID(1)).some.orElseFail(new Throwable("User not found"))  
		email = Email(user.email, Html("Hello here"))  
		_ <- EmailService.sendMail(email)  
		} yield ()  
	}  
}
}
case class User(id: UserID, email: EmailAddress)  
case class UserID(id: Int) extends AnyVal
```

Инфраструктурные зависимости имеет смысл выносить на уровень сигнатуры, бизнесовые зависимости лучше не выносить их можно получать в качестве экземпляров конструкторов.

```java
trait Service{  
	def notifyUser(id: UserID): RIO[EmailService with Console, Unit]  
}  
  
class ServiceImpl(userDAO: UserDAO.Service) extends Service{  
	override def notifyUser(id: UserID): RIO[EmailService with Console, Unit] = for{  
	emailService <- ZIO.environment[EmailService].map(_.get)  
	user <- userDAO.findBy(UserID(1)).some.orElseFail(new Throwable("User not found"))  
	email = Email(user.email, Html("Hello here"))  
	_ <- emailService.sendMail(email)  
	} yield ()  
}
```

Усовершенствуем доступ к EmailService

```java
class ServiceImpl(userDAO: UserDAO.Service) extends Service{  
	override def notifyUser(id: UserID): RIO[EmailService with Console, Unit] = for{  
	user <- userDAO.findBy(UserID(1)).some.orElseFail(new Throwable("User not found"))  
	email = Email(user.email, Html("Hello here"))  
	_ <- ZIO.accessM[EmailService with Console](_.get.sendMail(email))  
	} yield ()  
}
```

Везде где в имени есть 'M' то мы либо принимаем лямбду возвращающую Zio, либо просто оперируем в терминах Zio

Можно создать прокси метод в EmailService и использовать его напрямую
```java
object EmailService {
	def sendMail(email: Email): URIO[EmailService with zio.console.Console, Unit] = ZIO.accessM(_.get.sendMail(email))
}

class ServiceImpl(userDAO: UserDAO.Service) extends Service{  
	override def notifyUser(id: UserID): RIO[EmailService with Console, Unit] = for{  
	user <- userDAO.findBy(UserID(1)).some.orElseFail(new Throwable("User not found"))  
	email = Email(user.email, Html("Hello here"))  
	_ <- EmailService.sendMail(email)  
	} yield ()  
}
```

Такие прокси сервисы можно создавать автоматически через анотацию @accessible 
```java
@accessible 
object EmailService {
	...
}
```

Соберем слой в UserService
```java
object UserService{  
	... 
	val live: ZLayer[UserDAO, Nothing, UserService] =  
	ZLayer.fromService[UserDAO.Service, UserService.Service](dao => new ServiceImpl(dao))  
}
```

## Zio сервис паттерн
Композиция сервисов
- Вертикальная
 ![[verticalCompositionZlayer.png|200]]
![[gorizontalCompositionLayer.png|300]]

- Горизонтальная
![[girizontalCompositionZlayer.png|300]]
![[verticalCompositionLayer.png|300]]

- Mix
![[mixCompositionLayer.png|300]]

Запуск
```java
object ZioApp extends zio.App{  
  
	lazy val env: ZLayer[Any, Nothing, UserService with EmailService] = UserDAO.live >>> UserService.live ++ EmailService.live  
	  
	lazy val app: ZIO[UserService with EmailService with Console, Throwable, Unit] = UserService.notifyUser(UserID(10))  
	  
	override def run(args: List[String]): URIO[zio.ZEnv, ExitCode] =  
	app.provideSomeLayer[Console](env).exitCode  
}
```
