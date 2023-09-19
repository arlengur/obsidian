## Base
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

Каждый Request/Response состоит из 2х частей
- Метаданные (заголовки) - доступны сразу
- Тело (отправленные данные) - материализуется лениво

EntityEncoder/EntityDecoder - кодеки выполняющие материализацию и поэтому зависят от IO

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

# Middleware

## Header
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

## Session
```scala
import cats.effect.{IO, IOApp, Ref, Resource}  
import com.comcast.ip4s.{Host, Port}  
import org.http4s.HttpRoutes  
import org.http4s.dsl.io._  
import org.http4s.ember.server.EmberServerBuilder  
import org.http4s.server.Router  
import org.typelevel.ci.CIStringSyntax  
  
object Restfull {  
	def routerSessions(sessions: Sessions[IO]) = Router("/" -> serviceSessions(sessions))  
	  
	type Sessions[F[_]] = Ref[F, Set[String]]  
	  
	def serviceSessions(sessions: Sessions[IO]): HttpRoutes[IO] =  
	HttpRoutes.of {  
		case r@GET -> Root / "hello" =>  
			r.headers.get(ci"X-User-Name") match {  
				case Some(values) =>  
					val name = values.head.value  
					sessions.get.flatMap(users =>  
						if (users.contains(name)) Ok(s"Hello, $name")  
						else Forbidden(s"$name shall not pass!!!")  
					)  
				case None => Forbidden("empty shall not pass!!!")  
			}  
		case PUT -> Root / "login" / name =>  
			sessions.update(set => set + name).flatMap(_ => Ok("done"))  
	}  
	  
	val server = for {  
		sessions <- Resource.eval(Ref.of[IO, Set[String]](Set.empty))  
		s <- EmberServerBuilder  
			.default[IO]  
			.withPort(Port.fromInt(8080).get)  
			.withHost(Host.fromString("localhost").get)  
			.withHttpApp(routerSessions(sessions).orNotFound).build  
	} yield s  
}  
  
object main extends IOApp.Simple {  
	def run(): IO[Unit] = {  
		Restfull.server.use(_ => IO.never)  
	}  
}
```

Более модульно
```scala
import cats.data.{Kleisli, OptionT}  
import cats.effect.{IO, IOApp, Ref, Resource}  
import com.comcast.ip4s.{Host, Port}  
import org.http4s.HttpRoutes  
import org.http4s.dsl.io._  
import org.http4s.ember.server.EmberServerBuilder  
import org.http4s.server.{HttpMiddleware, Router}  
import org.typelevel.ci.CIStringSyntax  
import cats.implicits._  
  
object Restfull {  
	def LoginService(sessions: Sessions[IO]): HttpRoutes[IO] =  
	HttpRoutes.of {  
		case PUT -> Root / "login" / name =>  
			sessions.update(set => set + name).flatMap(_ => Ok("done"))  
	}  
	  
	def ServiceHello: HttpRoutes[IO] =  
	HttpRoutes.of {  
		case r@GET -> Root / "hello" =>  
			r.headers.get(ci"X-User-Name") match {  
		case Some(values) =>  
			val name = values.head.value  
			Ok(s"Hello, $name")  
		case None => Forbidden("you shall not pass!!!")  
		}  
	}  
	  
	type Sessions[F[_]] = Ref[F, Set[String]]  
	  
	def ServiceAuth(sessions: Sessions[IO]): HttpMiddleware[IO] =  
	routes =>  
	Kleisli { req =>  
		req.headers.get(ci"X-User-Name") match {  
			case Some(values) =>  
				val name = values.head.value  
				for {  
					users <- OptionT.liftF(sessions.get)  
					results <-  
					if (users.contains(name)) routes(req)  
					else OptionT.liftF(Forbidden(s"$name shall not pass!!!"))  
				} yield results  
			case None => OptionT.liftF(Forbidden("empty shall not pass!!!"))  
		}  
	}  

	def routerSessionAuth(sessions: Sessions[IO]) =  
	Router("/" -> (LoginService(sessions) <+> ServiceAuth(sessions)(ServiceHello)))  
	  
	val server = for {  
		sessions <- Resource.eval(Ref.of[IO, Set[String]](Set.empty))  
		s <- EmberServerBuilder  
			.default[IO]  
			.withPort(Port.fromInt(8080).get)  
			.withHost(Host.fromString("localhost").get)  
			.withHttpApp(routerSessionAuth(sessions).orNotFound).build 
	} yield s  
}  
  
object main extends IOApp.Simple {  
	def run(): IO[Unit] = {  
		Restfull.server.use(_ => IO.never)  
	}  
}
```

Production style
```scala
import cats.data.{Kleisli, OptionT}  
import cats.effect.{IO, IOApp, Ref, Resource}  
import cats.implicits._  
import com.comcast.ip4s.{Host, Port}  
import org.http4s.dsl.io._  
import org.http4s.ember.server.EmberServerBuilder  
import org.http4s.server.{AuthMiddleware, Router}  
import org.http4s.{AuthedRequest, AuthedRoutes, HttpRoutes}  
import org.typelevel.ci.CIStringSyntax  
  
object Restfull {  
	def LoginService(sessions: Sessions[IO]): HttpRoutes[IO] =  
	HttpRoutes.of {  
		case PUT -> Root / "login" / name =>  
		sessions.update(set => set + name).flatMap(_ => Ok("done"))  
	}  
	  
	def ServiceHelloAuth: AuthedRoutes[User, IO] = AuthedRoutes.of {  
		case GET -> Root / "hello" as user =>  
		Ok(s"Hello, ${user.name}")  
	}  
	  
	type Sessions[F[_]] = Ref[F, Set[String]]  
	  
	final case class User(name: String)  
	  
	def ServiceAuthMiddleware(sessions: Sessions[IO]): AuthMiddleware[IO, User] =  
	routes =>  
		Kleisli { req =>  
			req.headers.get(ci"X-User-Name") match {  
			case Some(values) =>  
				val name = values.head.value  
				for {  
					users <- OptionT.liftF(sessions.get)  
					results <-  
					if (users.contains(name)) routes(AuthedRequest(User(name), req))  
					else OptionT.liftF(Forbidden(s"$name shall not pass!!!"))  
				} yield results  
			case None => OptionT.liftF(Forbidden("empty shall not pass!!!"))  
		}  
	}  
	  
	def routerSessionAuthProd(sessions: Sessions[IO]) =  
	Router("/" -> (LoginService(sessions) <+> ServiceAuthMiddleware(sessions)(ServiceHelloAuth)))  
	  
	val server = for {  
		sessions <- Resource.eval(Ref.of[IO, Set[String]](Set.empty))  
		s <- EmberServerBuilder  
			.default[IO]  
			.withPort(Port.fromInt(8080).get)  
			.withHost(Host.fromString("localhost").get)  
			.withHttpApp(routerSessionAuthProd(sessions).orNotFound).build  
	} yield s  
}  
  
object main extends IOApp.Simple {  
	def run(): IO[Unit] = {  
		Restfull.server.use(_ => IO.never)  
	}  
}
```

## Testing
```scala
object Test extends IOApp.Simple {  
	import Restfull.{User, ServiceHelloAuth}  
	def run: IO[Unit] = {  
		for {  
			result <- ServiceHelloAuth(AuthedRequest(User("abc"), Request(method = Method.GET,  
			uri = Uri.fromString("/hello").toOption.get))).value  
			_ <- result match {  
				case Some(resp) => resp.bodyText.compile.last.flatMap(IO.println)  
				case None => IO.println("fail")  
			}  
		} yield ()  
	}  
}
```

## Client 
```scala
object HttpClient {  
	val builder: Resource[IO, Client[IO]] = EmberClientBuilder.default[IO].build  
	val request: Request[IO] = Request[IO](uri = Uri.fromString("http://localhost:8080/hello").toOption.get)  
	  
	val result: Resource[IO, Response[IO]] = for {  
		client <- builder  
		response <- client.run(request)  
	} yield response  
}  
  
object main extends IOApp.Simple {  
	def run(): IO[Unit] = for {  
		fiber <- Restfull.server.use(_ => IO.never).start  
		_ <- HttpClient.result.use(IO.println)  
		_ <- fiber.join  
	} yield ()  
}
```

Response является потоком, его можно прочитать только 1 раз 

Обработка ответа на клиенте
```scala
val result: Resource[IO, String] = for {  
	client <- builder  
	response <- effect.Resource.eval(client.expect[String](request))  
} yield response
```

Получить текст ответа на клиенте
```scala
object HttpClient {  
	val builder: Resource[IO, Client[IO]] = EmberClientBuilder.default[IO].build  
	val request: Request[IO] = Request[IO](uri = Uri.fromString("http://localhost:8080/hello").toOption.get)  
	  
	val result = builder.use(  
		client => client.run(request).use(  
			resp => if (!resp.status.isSuccess)  
			resp.body.compile.to(Array).map(new String(_))  
			else  
			IO("Error")  
		)  
	)  
}  
  
object main extends IOApp.Simple {  
	def run(): IO[Unit] = for {  
		_ <- Restfull1.serverSessionsAuthClear.use(_ => HttpClient.result.flatMap(IO.println) *> IO.never)  
	} yield ()  
}
```

# Circe
```scala
import cats.effect.{IO, IOApp, Resource}  
import com.comcast.ip4s.{Host, Port}  
import http4smiddleware.Restfull.User  
import io.circe.derivation.{deriveDecoder, deriveEncoder}  
import io.circe.{Decoder, Encoder}  
import org.http4s.circe.CirceEntityDecoder._  
import org.http4s.circe.CirceEntityEncoder._  
import org.http4s.client.Client  
import org.http4s.dsl.io._  
import org.http4s.ember.client.EmberClientBuilder  
import org.http4s.ember.server.EmberServerBuilder  
import org.http4s.server.Router  
import org.http4s.{HttpRoutes, Method, Request, Uri}  
  
object Restfull {  
	case class User(name: String, email: Option[String])  
	  
	implicit val decoderUser: Decoder[User] = deriveDecoder  
	implicit val encoderUser: Encoder[User] = deriveEncoder  
	  
	def publicRoutes: HttpRoutes[IO] = HttpRoutes.of {  
		case r@POST -> Root / "echo" =>  
			for {  
				user <- r.as[User]  
				response <- Ok(user)  
			} yield response  
	}  
	  
	def router = Router("/" -> publicRoutes)  
	  
	val server = for {  
		s <- EmberServerBuilder  
			.default[IO]  
			.withPort(Port.fromInt(8080).get)  
			.withHost(Host.fromString("localhost").get)  
			.withHttpApp(router.orNotFound).build  
	} yield s  
}  
  
object HttpClient {  
  
	val builder: Resource[IO, Client[IO]] = EmberClientBuilder.default[IO].build  
	val request: Request[IO] = Request[IO](  
	method = Method.POST,  
	uri = Uri.fromString("http://localhost:8080/echo").toOption.get)  
	.withEntity(User("arlen", Some("satovritti@gmail.com")))  
	  
	val result: IO[User] = builder.use(  
		client => client.run(request).use(  
			resp => if (resp.status.isSuccess)  
			resp.as[User]  
		else  
			IO.raiseError(new Exception("error"))  
		)  
	)  
}  
  
object main extends IOApp.Simple {  
	def run(): IO[Unit] = for {  
		_ <- Restfull.server.use(_ => HttpClient.result.flatMap(IO.println) *> IO.never)  
	} yield ()  
}
```