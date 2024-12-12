
```scala
package com.qiwi.ads.marmot_day  
  
import com.qiwi.ads.dm.protocol.Message  
import com.qiwi.ads.marmot_day.DateTimeUtils.dtFormatSec  
import com.qiwi.ads.marmot_day.TestUtils.makeRandomString  
import io.circe.Json  
import io.circe.syntax._  
  
import java.io.FileWriter  
import java.time.{LocalDateTime, ZoneOffset}  
import scala.util.Random  
  
object MessageDataGenerator extends App {  
  val start          = LocalDateTime.of(2021, 1, 1, 0, 0)  
  val end            = LocalDateTime.of(2021, 1, 2, 0, 0)  
  val step           = 5  
  val outputFileName = "marmot_day.tsv"  
  
  Iterator  
    .iterate(start)(_.plusSeconds(step))  
    .takeWhile(end.isAfter)  
    .toSeq  
    .map(time =>  
      Message(  
        Random.nextInt(),  
        "20",  
        makeRandomString(5),  
        time.toInstant(ZoneOffset.UTC).toEpochMilli,  
        "Europe/Moscow",  
        Map(  
          "TXN_DATE" -> Json.fromString(time.format(dtFormatSec))  
        ),  
      )  
    )  
    .map(_.asJson.noSpaces)  
    .foreach { str =>  
      val fw = new FileWriter(outputFileName, true)  
      try {  
        fw.write(str + System.lineSeparator)  
      } finally {  
        fw.flush()  
        fw.close()  
      }  
    }  
}
```
