```scala
package com.qiwi.ads.marmot_day  
  
import zhttp.http._  
import zhttp.service.Server  
import zio._  
  
object DmServerTest extends ZIOAppDefault {  
  val post: Http[Any, Nothing, Request, Response] =  
    Http.collect[Request] { case Method.POST -> !! => Response.ok}  
  
  def run: ZIO[Any, Throwable, Unit] = for {  
    _ <- Console.printLine(s"Starting server on http://localhost:8080")  
    _ <- Server.start(8080, post)  
  } yield ()
  
}
```
