Экосистема модулей для работы с HTTP, построенная на базе Cats и FS2
Основные модули
- core
- dsl
- server
- client
- blaze - IO бэкенд на основе netty
- ember - IO бэкенд на основе fs2 & nio (с сокетами)

type Http[F[_]] = Kleisli[F, Request, Response]
Request => F[Response]

type HttpRoutes[F[_]] = Http[OptionT[F, *], F]
Request => F[Option[Response]]

Зависимости
```
libraryDependencies += "org.http4s" %% "http4s-client" % "0.23.18"  
libraryDependencies += "org.http4s" %% "http4s-dsl" % "0.23.18"  
libraryDependencies += "org.http4s" %% "http4s-ember-server" % "0.23.18"  
libraryDependencies += "org.http4s" %% "http4s-ember-client" % "0.23.18"
```

Пример
```scala
object Restfull {  
	val service: HttpRoutes[IO] =  
		HttpRoutes.of {  
			case GET -> Root / "hello"/ name => Ok(name)  
	}  
	  
	val httpApp = service.orNotFound  
	val server = for {  
		s <- EmberServerBuilder  
		.default[IO]  
		.withPort(Port.fromInt(8080).get)  
		.withHost(Host.fromString("localhost").get)  
		.withHttpApp(httpApp).build  
	} yield s  
}  
  
object mainServer extends IOApp.Simple{  
	def run(): IO[Unit] ={  
		Restfull.server.use(_ => IO.never)  
	}  
}
```

Пример
```scala
import cats.effect.{IO, IOApp}  
import com.comcast.ip4s.{Host, Port}  
import org.http4s.HttpRoutes  
import org.http4s.dsl.io._  
import org.http4s.ember.server.EmberServerBuilder  
import org.http4s.server.Router

object Restfull {  
	val serviceOne: HttpRoutes[IO] =  
		HttpRoutes.of {  
			case GET -> Root / "hello" / name => Ok("from service one")  
		}  
	  
	val serviceTwo: HttpRoutes[IO] =  
		HttpRoutes.of {  
			case GET -> Root / "hello" / name => Ok("from service two")  
		}  
	  
	val router = Router("/" -> serviceOne, "/api" -> serviceTwo)  
	  
	val server = for {  
		s <- EmberServerBuilder  
		.default[IO]  
		.withPort(Port.fromInt(8080).get)  
		.withHost(Host.fromString("localhost").get)  
		.withHttpApp(router.orNotFound).build  
	} yield s  
}  
  
object mainServer extends IOApp.Simple{  
	def run(): IO[Unit] ={  
		Restfull.server.use(_ => IO.never)  
	}  
}
```

Middleware
```scala
def addHeaderMiddleware[F[_]: Functor](routes: HttpRoutes[F]): HttpRoutes[F] = Kleisli { req =>  
	val maybeResponse = routes(req)  
	  
	maybeResponse.map{  
		case Status.Successful(res) => res.putHeaders("X-Otus" -> "Hello")  
		case other => other  
	}  
}

val router = Router("/" -> serviceOne, "/api" -> addHeaderMiddleware(serviceTwo))
```
