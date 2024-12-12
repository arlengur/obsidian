```scala
package com.qiwi.ads.marmot_day  
  
import com.qiwi.ads.dm.protocol.Message  
import zio._  
  
import java.time.format.DateTimeFormatter  
import java.time.temporal.ChronoUnit  
import java.time.{Instant, LocalDateTime, LocalTime, OffsetDateTime, ZoneId}  
  
object DateTimeUtils {  
  val dtFormatSec: DateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")  
  
  def messageEmittingMs(message: Message, timezoneId: String): Long =  
    Instant  
      .ofEpochMilli(message.unixTimestampMs)  
      .atZone(ZoneId.of(timezoneId))  
      .toLocalTime  
      .atDate(LocalDateTime.now().toLocalDate)  
      .atZone(ZoneId.of(timezoneId))  
      .toInstant  
      .toEpochMilli  
  
  def messageEmittingDtTm(message: Message, currentDateTime: OffsetDateTime): LocalDateTime =  
    Instant  
      .ofEpochMilli(message.unixTimestampMs)  
      .atZone(ZoneId.of(message.timezoneId))  
      .toLocalTime  
      .atDate(currentDateTime.toLocalDate)  
  
  def getMillisUntilMidnight: ZIO[Any, Throwable, Long] =  
    Clock.currentDateTime.flatMap { curDtTm =>  
      ZIO.attempt(  
        ChronoUnit.MILLIS  
          .between(curDtTm.toLocalDateTime, LocalDateTime.of(curDtTm.toLocalDate.plusDays(1), LocalTime.MIDNIGHT))  
      )  
    }  
  
  def defineTimeToSleep(message: Message, maxMillisToSleep: Long): UIO[Long] =  
    for {  
      timeDiff <- timeDifference(message)  
      // Set a maximum sleep duration (maxMillisToSleep) to handle edge cases where the application is launched shortly  
      // before midnight. This avoids a discrepancy where transactions may have a time from the end of one day      // and a date from the beginning of the next, resulting in a misleading time difference calculation that      // could incorrectly span almost an entire 24-hour period.      // todo it should be a better way to handle this  
      res <- ZIO.succeed(if (timeDiff > maxMillisToSleep) 0L else timeDiff)  
    } yield res  
  
  def timeDifference(message: Message): UIO[Long] =  
    Clock.currentDateTime.map { currDtTm =>  
      message.unixTimestampMs - currDtTm.toInstant.toEpochMilli  
    }  
  
  def sleepIfRequired(message: Message, maxMillisToSleep: Long): UIO[Unit] =  
    for {  
      timeToSleep <- defineTimeToSleep(message, maxMillisToSleep)  
      _ <- ZIO.when(timeToSleep > 0)(  
             ZIO.logDebug(s"Sleeping $timeToSleep...") *> ZIO.sleep(timeToSleep.millisecond)  
           )  
    } yield ()  
  
  // if millisUntilMidnight > maxMillisBeforeNewDay we suppose that new  
  // day has already started and there is no need to sleep  private val maxMillisBeforeNewDay = 3600000  
  def sleepUntilMidnightIfRequired: IO[Throwable, Unit] =  
    for {  
      millisUntilMidnight     <- getMillisUntilMidnight  
      hasNewDayAlreadyStarted <- ZIO.succeed(millisUntilMidnight > maxMillisBeforeNewDay)  
      _ <- if (!hasNewDayAlreadyStarted)  
             ZIO.logInfo(s"Going to sleep $millisUntilMidnight millis before new day") *>  
               ZIO.sleep(millisUntilMidnight.millis)  
           else ZIO.logWarning("No need to sleep because new day has already started")  
    } yield ()  
  
}
```