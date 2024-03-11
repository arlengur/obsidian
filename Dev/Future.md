Нессылочно прозрачна
```scala
import scala.concurrent.ExecutionContext.Implicits.global  
import scala.concurrent.duration.DurationInt  
import scala.concurrent.{Await, Future}  
import scala.util.Random  
val future1 = {  
	val r = new Random(0L) // Initialize Random with a fixed seed
	// nextInt has the side-effect of moving to  
	// the next random number in the sequence:  
	val x = Future(r.nextInt)  
	for {  
		a <- x  
		b <- x  
	} yield (a, b)  
}  
val future2 = {  
	val r = new Random(0L)  
	for {  
		a <- Future(r.nextInt)  
		b <- Future(r.nextInt)  
	} yield (a, b)  
}  
val result1 = Await.result(future1, 1.second) // (-1155484576, -1155484576)  
val result2 = Await.result(future2, 1.second) // (-1155484576, -723955400)
```
Тут future1 вызывает nextInt 1 раз, а future2 - 2 раза.

Так же Future запускается сразу, в момент объявления