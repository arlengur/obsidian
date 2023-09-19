Reactive Streams
- Издатель - асинхронно создает и отправляет в поток элементы (Source)
- Подписчик - получает элементы (Sink)
- Обработчик - трансформирует элементы (Flow)
- BackPressure - подписчик не успевает считывать элементы

![[AkkaStreamsPipeline.png|400]]

Actor Materializer
- проверяет соединения между входами и выходами в графе
- создает акторы из Source и Sink
- сокеты, порты, пулы потоков

Процесс построения
- Source
	- `val source: Source[Int, NotUsed] = Source(1 to 5)`
- Flow
	- `val flow = Flow[Int].map(x => x + 1) `
- Sink
	- `val sink = Sink.foreach[Int](println)`
- Runnable Graph
	- `val runnable = source.via(flow).to(sink)`
- Run
	- `implicit val system: ActorSystem = ActorSystem("fusion")`
	- `implicit val materializer: ActorMaterializer = ActorMaterializer()`
	- `runnable.run()`

```scala
import akka.NotUsed  
import akka.actor.ActorSystem  
import akka.stream.ActorMaterializer  
import akka.stream.scaladsl.{Flow, Sink, Source}  
  
object Ex1 {  
	def main(args: Array[String]): Unit = {  
		implicit val system: ActorSystem = ActorSystem("fusion")  
		implicit val materializer: ActorMaterializer = ActorMaterializer()  
		  
		val source: Source[Int, NotUsed] = Source(1 to 5)  
		val flow = Flow[Int].map(x => x + 1)  
		val sink = Sink.foreach[Int](println)  
		  
		source.via(flow).to(sink).run()  
	}  
}
```

```scala
import akka.NotUsed  
import akka.actor.ActorSystem  
import akka.stream.ActorMaterializer  
import akka.stream.scaladsl.{Flow, Sink, Source}  
  
object Ex2 {  
	def main(args: Array[String]): Unit = {  
		implicit val system: ActorSystem = ActorSystem("fusion")  
		implicit val materializer: ActorMaterializer = ActorMaterializer()  
		  
		val source: Source[Int, NotUsed] = Source(1 to 5)  
		val flow1 = Flow[Int].map { x =>  
			Thread.sleep(1000)  
			x + 1  
		}  
		val flow2 = Flow[Int].map { x =>  
			Thread.sleep(1000)  
			x * 10  
		}  
		val sink = Sink.foreach[Int](println)  
		  
		source.async  
			.via(flow1).async  
			.via(flow2).async  
			.to(sink)  
			.run()  
	}  
}
```

## BackPressureEx
```scala
import akka.actor.ActorSystem  
import akka.stream.scaladsl.{Flow, Sink, Source}  
import akka.stream.{ActorMaterializer, OverflowStrategy}  
  
object BackPressureEx extends App {  
	val source = Source(1 to 10)  
	val flow = Flow[Int].map { el =>  
		println(s"Flow inside: $el")  
		el + 10  
	}  
	  
	val flowWithBuffer = flow.buffer(3, overflowStrategy = OverflowStrategy.dropHead)  
	val sink = Sink.foreach[Int] { el =>  
		Thread.sleep(1000)  
		println(s"Sink inside : $el")  
	}  
	  
	implicit val system: ActorSystem = ActorSystem("fusion")  
	implicit val materializer: ActorMaterializer = ActorMaterializer()  
	source  
		.via(flow).async  
		.to(sink)  
		// .run()  
	  
	source  
		.via(flowWithBuffer).async  
		.to(sink)  
		// .run()  
}
```

## Graph DSL
![](graphDSL.png)

Graph с двумя выходами
![](graphWithTwoSink.png)

Сбалансированный Graph
![](balancedGraph.png)

Проблемы
- не успеваем обработать много событий
- медленный компонент графа
- потеря сообщений
- запись в несколько получателей

Возможные способы решения
- асинхронная работа
- контроль отставания обработки элементов
- гарантии порядка обработки элементов
- Graph DSL

```scala
import akka.NotUsed  
import akka.actor.ActorSystem  
import akka.stream.{ActorMaterializer, ClosedShape}  
import akka.stream.scaladsl.{Broadcast, Flow, GraphDSL, RunnableGraph, Sink, Source, Zip}  
  
object AkkaStreamsGraph {  
	implicit val system: ActorSystem = ActorSystem("Graph")  
	implicit val materializer: ActorMaterializer = ActorMaterializer()  
	  
	val graph =  
	GraphDSL.create() { implicit builder: GraphDSL.Builder[NotUsed] => 
		import GraphDSL.Implicits._  
		  
		val input = builder.add(Source(1 to 5))  
		  
		val broadcast = builder.add(Broadcast[Int](2))  
		  
		val increment = builder.add(Flow[Int].map(x => x + 1))  
		val multiplier = builder.add(Flow[Int].map(x => x * 10))  
		  
		val zip = builder.add(Zip[Int, Int])  
		  
		val output = builder.add(Sink.foreach[(Int, Int)](println))  
		  
		input ~> broadcast  
		  
		broadcast.out(0) ~> increment ~> zip.in0  
		broadcast.out(1) ~> multiplier ~> zip.in1  
		  
		zip.out ~> output  
		  
		ClosedShape  
	}  
	  
	def main(args: Array[String]): Unit = {  
		RunnableGraph.fromGraph(graph).run()  
	}  
}
```