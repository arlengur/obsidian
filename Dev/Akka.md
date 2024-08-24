..- Модель акторов - модель построения асинхронных и распределительных приложений. Она позволяет масштабироваться как вертикально так и горизонтально.
- Актор - основная сущность в данной модели. Каждый актор можно представить как очередь сообщений, очень легковесную, уменьшенную до микроскопических размеров. Объект актора представляет собой легковесный поток, это функция из сообщения в поведение.

Актор имеет
- адрес
- протокол (набор обрабатываемых сообщений)
- поведение (то что делает в ответ на сообщение, поведение может меняться)
- состояние (меняется как реакция на сообщение)
- жизненный цикл 

![](actor_lifecycle_1%2010.svg)

Свойства акторов:
- иерархии
- обмен сообщениями
- поведение
- состояние
- наблюдение
- независимость

ActorSystem - тяжеловесная конструкция, дает среду исполнения для наших акторов (одна на приложение)

Типы коммуникаций
- Отправил и забыл
	- Большая пропускная способность
	- может приводить к переполнению ящика
- Запрос / ответ
	- когда нужна реакция на отправленное сообщение
	- связанность между протоколами разных акторов
- Адаптер
	- необходим когда нужно обрабатывать ответы другого актора

Гарантии доставки сообщений
- At most once - отправляем сообщения и какие-то из них придут
- At least once - все что отправлено придет, но возможны дубликаты
- Exactly once - все что отправлено придет в единственном экземпляре (по умолчанию)

Порядок сообщений гарантируется в паре один отправитель - один получатель

Контекст позволяет 
- останавливать акторы
	- context.stop(childActorRef) - асинхронно останавливает актор (актор может какое-то время получать сообщения)
	- context.stop(self) - рекурсивно останавливает акторы
- наблюдать за акторами
	- context.watch(childActorRef) - если childActorRef то родитель получит сообщение Terminated

Актор можно остановить с помощью сообщений
- actorRef ! Kill - вызывает ActorKilledException
- actorRef ! PoisonPill - позволяет обработать все полученные сообщения

Экземпляр актора
- состояние
- бизнес методы
- поведение

Actor ref
- mailbox
- один экземпляр
- UUID

Для корректной работы необходимо подключить логгер
`"ch.qos.logback" % "logback-classic" % "1.2.3"`
 
## Функциональный и ООП стили акторов
```scala
// Функциональный стиль акторов
object func_style {  
	object Echo {  
		def apply(): Behavior[String] = Behaviors.setup { ctx =>  
			Behaviors.receiveMessage {  
				case msg =>  
					ctx.log.info(s"func: $msg")  
					Behaviors.same  
			}  
		}  
	}  
}

// ООП стиль акторов
object oop_style {  
	class Echo(ctx: ActorContext[String]) extends AbstractBehavior[String](ctx) {  
		def onMessage(msg: String): Behavior[String] = {  
			ctx.log.info(s"oop: $msg")  
			this  
		}  
	}  
	object Echo {  
		def apply(): Behavior[String] = Behaviors.setup { ctx =>  
			new Echo(ctx)  
		}  
	}  
}

def main(args: Array[String]): Unit = {  
	val system = ActorSystem[String](oop_style.Echo(), "func_actor")  
	system ! "Hello"  
	Thread.sleep(3000)  
	system.terminate()  
}
```

## Supervisor
Акторы можно создавать программно
```scala
import akka.actor.typed._  
import akka.actor.typed.scaladsl.AskPattern.{Askable, _}  
import akka.actor.typed.scaladsl.Behaviors  
import akka.util.Timeout  
  
import scala.concurrent.Future  
import scala.concurrent.duration.DurationInt  
import scala.language.postfixOps

object Supervisor {  
	def apply(): Behavior[SpawnProtocol.Command] = Behaviors.setup { ctx =>  
		ctx.log.info(ctx.self.toString)  
		SpawnProtocol()  
	}  
}  
  
def main(args: Array[String]): Unit = {  
	implicit val system: ActorSystem[SpawnProtocol.Command] = ActorSystem[SpawnProtocol.Command](Supervisor(), "Echo")  
	implicit val timeout: Timeout = Timeout(3 seconds)  
	  
	val echo: Future[ActorRef[String]] = system.ask(  
	SpawnProtocol.Spawn(func_style.Echo(), "Echo", Props.empty, _))  
	  
	implicit val ec = system.executionContext  
	for (ref <- echo)  
		ref ! "Hello from ask"  
}
```

## Изменение состояния
```scala
import akka.actor.typed.scaladsl.Behaviors  
import akka.actor.typed.{ActorSystem, Behavior}  
  
import scala.language.postfixOps  
  
object ChangeState {  
sealed trait WorkerProtocol  
  
object WorkerProtocol {  
	case object Start extends WorkerProtocol  
	case object StandBy extends WorkerProtocol  
	case object Stop extends WorkerProtocol  
}  
  
import WorkerProtocol._  
def apply(): Behavior[WorkerProtocol] = idle()  
  
def idle(): Behavior[WorkerProtocol] = Behaviors.setup { ctx =>  
	Behaviors.receiveMessage {  
	case msg@Start =>  
		ctx.log.info(msg.toString)  
		workInProgress()  
	case msg@StandBy =>  
		ctx.log.info(msg.toString)  
		idle()  
	case msg@Stop =>  
		ctx.log.info(msg.toString)  
		Behaviors.stopped  
	}  
}  
  
def workInProgress(): Behavior[WorkerProtocol] = Behaviors.setup { ctx =>  
	Behaviors.receiveMessage {  
		case Start =>  
			ctx.log.info("workInProgress started")  
			Behaviors.unhandled  
		case StandBy =>  
			ctx.log.info("go to standBy")  
			idle()  
		case Stop =>  
			ctx.log.info("stopped")  
			Behaviors.stopped  
	}  
}  
  
def main(args: Array[String]): Unit = {  
	implicit val system: ActorSystem[WorkerProtocol] = ActorSystem[ChangeState.WorkerProtocol](ChangeState(), "ChangeState")  
	  
	system ! Start  
	system ! StandBy  
	system ! Stop  
	}  
}
```

## Counter
```scala
import akka.actor.typed.scaladsl.AskPattern.Askable  
import akka.actor.typed.scaladsl.Behaviors  
import akka.actor.typed.{ActorRef, ActorSystem, Behavior, Scheduler}  
import akka.util.Timeout  
  
import scala.concurrent.Await  
import scala.concurrent.duration.DurationInt  
import scala.language.postfixOps  
  
object Counter {  
sealed trait CounterProtocol  
  
object CounterProtocol {  
	final case object Inc extends CounterProtocol  
	final case class GetCounter(replyto: ActorRef[Int]) extends CounterProtocol  
}  
  
import CounterProtocol._  
  
def apply(init: Int): Behavior[CounterProtocol] = inc(init)  
  
def inc(counter: Int): Behavior[CounterProtocol] = Behaviors.setup { _ =>  
	Behaviors.receiveMessage {  
		case Inc =>  
			inc(counter + 1)  
		case GetCounter(replyTo) =>  
			replyTo ! counter  
			Behaviors.same  
	}  
}  
  
def main(args: Array[String]): Unit = {  
	implicit val system: ActorSystem[CounterProtocol] = ActorSystem[CounterProtocol](Counter(0), "Counter")  
	implicit val timeout: Timeout = Timeout(3 seconds)  
	implicit val sc: Scheduler = system.scheduler  
	  
	system ! Inc  
	system ! Inc  
	val res = system ? GetCounter  
	println(Await.result(res, 1 seconds))  
	system.terminate()  
	}  
}
```

## Adapter template
```scala
import akka.NotUsed  
import akka.actor.typed.scaladsl.Behaviors  
import akka.actor.typed.{ActorRef, ActorSystem, Behavior}  
import ru.arlen.examples.Dispatcher.DispatcherCommand.{LogWork, ParseUrl, TaskDispatcher}  
import ru.arlen.examples.Dispatcher.LogWorker.{Log, LogResponse}  
import ru.arlen.examples.Dispatcher.ParseUrlWorker.{Parse, ParseResponse}  
  
object Dispatcher {  
	object DispatcherCommand {  
		sealed trait TaskDispatcher  
		case class ParseUrl(url: String) extends TaskDispatcher  
		case class LogWork(str: String) extends TaskDispatcher  
		case class LogResponseWrapper(msg: LogResponse) extends TaskDispatcher  
		case class ParseResponseWrapper(msg: ParseResponse) extends TaskDispatcher  
		  
		def apply(): Behavior[TaskDispatcher] = Behaviors.setup { ctx =>  
			val logAdapter = ctx.messageAdapter[LogResponse](rs => LogResponseWrapper(rs))  
			val parseAdapter = ctx.messageAdapter[ParseResponse](rs => ParseResponseWrapper(rs))  
			  
			Behaviors.receiveMessage {  
				case ParseUrl(url) =>  
					val ref = ctx.spawn(ParseUrlWorker(), s"ParseWorker-${java.util.UUID.randomUUID().toString}")  
					ctx.log.info(s"Dispatcher received url $url")  
					ref ! Parse(url, parseAdapter)  
					Behaviors.same  
				case LogWork(str) =>  
					val ref = ctx.spawn(LogWorker(), s"LogWorker-${java.util.UUID.randomUUID().toString}")  
					ctx.log.info(s"Dispatcher received log $str")  
					ref ! Log(str, logAdapter)  
					Behaviors.same  
				case LogResponseWrapper(_) =>  
					ctx.log.info("Log done")  
					Behaviors.same  
				case ParseResponseWrapper(_) =>  
					ctx.log.info("Parse done")  
					Behaviors.same  
			}  
		}  
	}  
object LogWorker {  
	sealed trait LogRequest  
	  
	case class Log(str: String, replyTo: ActorRef[LogResponse]) extends LogRequest  
	sealed trait LogResponse  
	case object LogDone extends LogResponse  
	  
	def apply(): Behavior[LogRequest] = Behaviors.setup { ctx =>  
		Behaviors.receiveMessage {  
			case Log(_, replyTo) =>  
				ctx.log.info("log work in progress")  
				replyTo ! LogDone  
				Behaviors.stopped  
		}  
	}  
}  
  
object ParseUrlWorker {  
	sealed trait ParseRequest  
	case class Parse(url: String, replyTo: ActorRef[ParseResponse]) extends ParseRequest  
	sealed trait ParseResponse  
	case object ParseDone extends ParseResponse  
	  
	def apply(): Behavior[ParseRequest] = Behaviors.setup { ctx =>  
		Behaviors.receiveMessage {  
			case Parse(_, replyTo) =>  
				ctx.log.info("parse work in progress")  
				replyTo ! ParseDone  
				Behaviors.stopped  
		}  
	}  
}  
  
def apply(): Behavior[NotUsed] =  
	Behaviors.setup { ctx =>  
		val dispatcher = ctx.spawn(DispatcherCommand(), "dispatcher")  
		  
		dispatcher ! LogWork("log line")  
		dispatcher ! ParseUrl("parse url")  
		  
		Behaviors.same  
	}  
  
def main(args: Array[String]): Unit = {  
	implicit val system: ActorSystem[NotUsed] = ActorSystem(Dispatcher(), "dispatcher_actor_system")  
	
	// или 2й вариант запуска
	// implicit val system: ActorSystem[TaskDispatcher] = ActorSystem[TaskDispatcher](DispatcherCommand(), "dispatcher_actor_system")  
	//  
	// system ! LogWork("log line")  
	// system ! ParseUrl("parse url")  
	  
	Thread.sleep(3000)  
	system.terminate()  
	}  
}
```

## MutableState
Пример работы с разными состояниями
```scala
import akka.NotUsed  
import akka.actor.typed.scaladsl.Behaviors  
import akka.actor.typed.{ActorSystem, Behavior}  
  
object MutableState {  
	sealed trait Command  
	case class Deposite(v: Integer) extends Command  
	case class Withdraw(v: Int) extends Command  
	case class Get() extends Command  
	  
	object Account {  
		def apply(m: Int): Behavior[Command] = Behaviors.setup { ctx =>  
		var amount: Int = m  
		  
		Behaviors.receiveMessage {  
			case Deposite(v) =>  
				amount = amount + v  
				ctx.log.info(s"Deposite $v to amount $amount. Total state is $amount")  
				Behaviors.same  
			case Withdraw(v) =>  
				amount = amount - v  
				ctx.log.info(s"Withdraw $v from amount $amount. Total state is $amount")  
				Behaviors.same  
			case Get() =>  
				ctx.log.info(s"Total state is $amount")  
				Behaviors.same  
		}  
	}  
}  
  
def apply(): Behavior[NotUsed] =  
	Behaviors.setup { ctx =>  
		val account1 = ctx.spawn(Account(2000), "account_state_actor1")  
		val account2 = ctx.spawn(Account(42), "account_state_actor2")  
		  
		account1 ! Get()  
		account1 ! Deposite(1)  
		account2 ! Get()  
		account1 ! Withdraw(1)  
		  
		Behaviors.same  
	}  
  
	def main(args: Array[String]): Unit = {  
		implicit val system: ActorSystem[NotUsed] = ActorSystem(MutableState(), "two_accounts")  
		Thread.sleep(5000)  
		system.terminate()  
	}  
}
```

## Start-stop
```scala
import akka.NotUsed  
import akka.actor.typed.scaladsl.Behaviors  
import akka.actor.typed.{ActorRef, ActorSystem, Behavior}  
  
sealed trait Command  
  
case class StartChild(name: String) extends Command  
case class SendMessageToChild(name: String, msg: String, num: Int) extends Command  
case class StopChild(name: String) extends Command  
case object Stop extends Command  
  
object Parent {  
	def apply(): Behavior[Command] = withChildren(Map())  
	  
	def withChildren(childs: Map[String, ActorRef[Command]]): Behavior[Command] =  
		Behaviors.setup { ctx =>  
			Behaviors.receiveMessage {  
				case StartChild(name) =>  
					ctx.log.info(s"Start child $name")  
					val newChild = ctx.spawn(Child(), name)  
					withChildren(childs + (name -> newChild))  
				case msg@SendMessageToChild(name, _, i) =>  
					ctx.log.info(s"Send message to child $name num=$i")  
					val childOption = childs.get(name)  
					childOption.foreach(childRef => childRef ! msg)  
					Behaviors.same  
				case StopChild(name) =>  
					ctx.log.info(s"Stopping child with name $name")  
					val childOption = childs.get(name)  
					childOption.foreach(childRef => ctx.stop(childRef))  
					Behaviors.same  
				case Stop =>  
					ctx.log.info("Stopped parent")  
					Behaviors.stopped  
			}  
		}  
}  
  
object Child {  
	def apply(): Behavior[Command] = Behaviors.setup { ctx =>  
		Behaviors.receiveMessage { msg =>  
			ctx.log.info(s"Child got message $msg")  
			Behaviors.same  
		}  
	}  
}  
  
object StartStop extends App {  
	def apply(): Behavior[NotUsed] =  
		Behaviors.setup { ctx =>  
			val parent = ctx.spawn(Parent(), "parent")  
			parent ! StartChild("child1")  
			parent ! SendMessageToChild("child1", "message to child1", 0)  
			parent ! StopChild("child1")  
			parent ! SendMessageToChild("child1", "message to child1", 1)  
			Behaviors.same  
		}  
	  
	implicit val system: ActorSystem[NotUsed] = ActorSystem(StartStop(), "start_stop")  
	Thread.sleep(5000)  
	system.terminate()  
}
```
## Тестирование
```scala
import akka.actor.{ActorRef, ActorSystem}  
import akka.testkit.{ImplicitSender, TestActors, TestKit}  
import org.scalatest.BeforeAndAfterAll  
import org.scalatest.wordspec.AnyWordSpecLike  
  
class MySpec() extends TestKit(ActorSystem("MySpec"))  
	with ImplicitSender  
	with AnyWordSpecLike  
	with BeforeAndAfterAll {  
	"An echo actor" should {  
		"send message" in {  
			val echo: ActorRef = system.actorOf(TestActors.echoActorProps)  
			echo ! "hello"  
			expectMsg("hello")  
		}  
	}  
	  
	override def afterAll(): Unit = {  
		TestKit.shutdownActorSystem(system)  
	}  
}
```

## Start-stop processing
```scala
import akka.NotUsed  
import akka.actor.typed.scaladsl.Behaviors  
import akka.actor.typed.{ActorSystem, Behavior, DeathPactException, SupervisorStrategy}  
  
sealed trait Msg  
case class Fail(text: String) extends Msg  
case class Hello(text: String) extends Msg  
  
object Worker {  
	def apply(): Behavior[Msg] =  
		Behaviors.receiveMessage {  
			case Fail(text) =>  
				throw new RuntimeException(text)  
			case Hello(text) =>  
				println(text)  
				Behaviors.same  
		}  
}  
  
object MiddleManagement {  
	def apply(): Behavior[Msg] =  
		Behaviors.setup[Msg] { context =>  
			context.log.info("Middle Management starting up")  
			val child = context.spawn(Worker(), "child")  
			context.watch(child)  
			  
			Behaviors.receiveMessage[Msg] { message =>  
				child ! message  
				Behaviors.same  
			}  
		}  
}  
  
object Boss {  
	def apply(): Behavior[Msg] =  
	Behaviors.supervise(  
		Behaviors.setup[Msg] { ctx =>  
			ctx.log.info("Boss starting up")  
			val middleManagement = ctx.spawn(MiddleManagement(), "middle-management")  
			ctx.watch(middleManagement)  
			  
			Behaviors.receiveMessage[Msg] { message =>  
				middleManagement ! message  
				Behaviors.same  
			}  
		}  
	).onFailure[DeathPactException](SupervisorStrategy.restart)  
}  
  
object StartStopProcessing extends App {  
	def apply(): Behavior[NotUsed] =  
		Behaviors.setup { ctx =>  
			val boss = ctx.spawn(Boss(), "boss-management")  
			boss ! Hello("hi 1")  
			boss ! Fail("ping")  
			Thread.sleep(1000)  
			  
			boss ! Hello("hi 2")  
			  
			Behaviors.same  
		}  
		  
	implicit val system: ActorSystem[NotUsed] = ActorSystem(StartStopProcessing(), "akka_typed")  
	Thread.sleep(5000)  
	system.terminate()  
}
```