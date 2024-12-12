```scala
object Test extends ZIOAppDefault {  
  val source =  
    ZStream  
      .iterate(1)(_ + 1)  
      .take(5)  
      .tap(x =>  
        printLine(s"Producing Element $x")  
          .schedule(Schedule.duration(1.seconds).jittered)  
      )  
  
  val sink: ZSink[Any, IOException, Chunk[Int], Nothing, Unit] =  
    ZSink.foreach((e: Chunk[Int]) =>  
      printLine(s"Processing batch of events: $e")  
        .schedule(Schedule.duration(3.seconds).jittered)  
    )  
  
  val myApp =  
    source.transduce(ZSink.collectAllN[Int](5)).run(sink)  
  
  val myApp2 =  
    source.aggregateAsync(ZSink.collectAllN[Int](256)).run(sink)  
  
  override def run: ZIO[ZIOAppArgs, Any, Any] =  
    source  
      .mapZIOPar(10)(ZIO.succeed(_)).tap(x => printLine("map").schedule(Schedule.duration(5.seconds)))  
      .runCollect  
}
```

build.sbt
```
import Dependencies._  
import ScalaOptions._  
  
lazy val root = (project in file("."))  
  .enablePlugins(PackPlugin)  
  .settings(  
    name                     := "ao-ads-marmot-day",  
    organization             := "com.qiwi.ads",  
    scalaVersion             := "2.13.11",  
    scalacOptions            := scalaCompilerOptions,  
    libraryDependencies     ++= zio ++ scalaXml ++ sttp ++ logback ++ logstash ++ dmProtocol ++ tests,  
    dependencyOverrides     ++= overrides,  
    coverageOutputTeamCity   := true,  
    testFrameworks           += new TestFramework("zio.test.sbt.ZTestFramework"),  
    coverageExcludedPackages := "com.qiwi.ads.marmot_day.config.*;com.qiwi.ads.marmot_day.metrics.*",  
    commands += Command.command("verify") { state =>  
      "clean" ::  
        "scalafmtCheck" ::  
        "coverage" ::  
        "compile" ::  
        "test" ::  
        "coverageReport" ::  
        state  
    },  
  )
```

DateTimeUtils
```
package com.qiwi.ads.marmot_day  
  
import zio._  
  
import java.time.format.DateTimeFormatter  
import java.time.temporal.ChronoUnit  
import java.time.{Instant, LocalDate, LocalDateTime, ZoneId}  
  
object DateTimeUtils {  
  val dtFormatSec: DateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")  
  val dtFormatMs: DateTimeFormatter  = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss.SSS")  
  
  def millisUntilMidnight(instNow: Instant = Instant.now()): Long =  
    instNow.plus(1, ChronoUnit.DAYS).truncatedTo(ChronoUnit.DAYS).toEpochMilli - instNow.toEpochMilli  
  
  def messageDateTime(unixTimestampMs: Long, zoneId: String): LocalDateTime =  
    Instant  
      .ofEpochMilli(unixTimestampMs)  
      .atZone(ZoneId.of(zoneId))  
      .toLocalDateTime  
  
  def messageEmittingDtTm(  
    unixTimestampMs: Long,  
    zoneId: String,  
    currDate: LocalDate,  
  ): LocalDateTime =  
    Instant  
      .ofEpochMilli(unixTimestampMs)  
      .atZone(ZoneId.of(zoneId))  
      .toLocalTime  
      .atDate(currDate)  
  
  def messageEmittingMs(  
    unixTimestampMs: Long,  
    zoneId: String,  
    currDate: LocalDate,  
  ): Long =  
    Instant  
      .ofEpochMilli(unixTimestampMs)  
      .atZone(ZoneId.of(zoneId))  
      .toLocalTime  
      .atDate(currDate)  
      .atZone(ZoneId.of(zoneId))  
      .toInstant  
      .toEpochMilli  
  
  def timeDifference(  
    timestampMs: Long,  
    currInst: Instant,  
  ): Long = currInst.toEpochMilli - timestampMs  
  
//  def timeDifferenceMs(  
//    timestampMs: Long,  
//    zoneId: String,  
//    currDtTm: LocalDateTime,  
//  ): Long = currDtTm.atZone(ZoneId.of(zoneId)).toInstant.toEpochMilli - timestampMs  
  
  def defineTimeToSleep(  
    unixTimestamp: Long,  
    maxMillisToSleep: Long,  
    curInst: Instant,  
  ): Long = {  
    val timeDiff = unixTimestamp - curInst.toEpochMilli  
    // Set a maximum sleep duration (maxMillisToSleep) to handle edge cases where the application is launched shortly  
    // before midnight. This avoids a discrepancy where transactions may have a time from the end of one day    // and a date from the beginning of the next, resulting in a misleading time difference calculation that    // could incorrectly span almost an entire 24-hour period.    // todo it should be a better way to handle this    if (timeDiff > maxMillisToSleep) 50L else timeDiff  
  }  
  
  // if millisUntilMidnight > maxMillisBeforeNewDay we suppose that new  
  // day has already started and there is no need to sleep  private val maxMillisBeforeNewDay = 3600000  
  def sleepUntilMidnightIfRequired: IO[Throwable, Unit] =  
    for {  
      msUntilMidnight         <- ZIO.succeed(millisUntilMidnight())  
      hasNewDayAlreadyStarted <- ZIO.succeed(msUntilMidnight > maxMillisBeforeNewDay)  
      _ <- if (!hasNewDayAlreadyStarted)  
             ZIO.logInfo(s"Going to sleep $msUntilMidnight millis before new day") *>  
               ZIO.sleep(msUntilMidnight.millis)  
           else ZIO.logWarning("No need to sleep because new day has already started")  
    } yield ()  
  
}
```

HttpRequestsService
```
package com.qiwi.ads.marmot_day.services  
  
import com.qiwi.ads.dm.protocol.Message  
import com.qiwi.ads.marmot_day.adts.{FailedResponse, NonSucceedResponse, SttpResponse, SucceedResponse}  
import com.qiwi.ads.marmot_day.config.{AppConf, PrometheusConf}  
import com.qiwi.ads.marmot_day.metrics.{errorCounter, payloadCounter, timer}  
import sttp.capabilities  
import sttp.capabilities.zio.ZioStreams  
import sttp.client3.SttpClientException.{ConnectException, ReadException, TimeoutException}  
import sttp.client3.{Identity, RequestT, Response, SttpBackend, basicRequest}  
import sttp.model.Uri  
import zio._  
import zio.macros.accessible  
  
@accessible  
trait HttpRequestsService {  
  def makeHttpRequest(  
    requestUri: Uri,  
    message: Message,  
    headers: Map[String, String],  
  ): Task[SttpResponse]  
}  
  
class HttpRequestsServiceImpl(  
  appConf: AppConf,  
  prometheusConf: PrometheusConf,  
  backend: SttpBackend[Task, ZioStreams with capabilities.WebSockets],  
) extends HttpRequestsService {  
  override def makeHttpRequest(  
    requestUri: Uri,  
    message: Message,  
    headers: Map[String, String],  
  ): Task[SttpResponse] =  
    for {  
      curInst <- Clock.instant  
      req     <- buildRequest(requestUri, msgToXML(message), headers, message.messageTypeId)  
//      _       <- makePauseForSpreading()  
//      _ <- ZIO  
//             .attempt(timeDifference(message.unixTimestampMs, curInst))  
//             .flatMap(delay => emittingDelayHist.update(delay.toDouble))  
      res <- sendRequest(req, message)  
    } yield SucceedResponse(Response.ok(""))  
  
  private def msgToXML: Message => String =  
    msg => <IRIS Version="1" Message="ModelRequest" MessageTypeId={msg.messageTypeId} MessageId={msg.messageId}>  
      {for { (tagName, value) <- msg.payload } yield <xml>{value.asString.getOrElse("_")}</xml>.copy(label = tagName)}  
    </IRIS>.toString  
  
  private def buildRequest(  
    requestUri: Uri,  
    payload: String,  
    headers: Map[String, String],  
    transactionMtid: String,  
  ): UIO[RequestT[Identity, Either[String, String], Any]] =  
    ZIO.succeed(  
      basicRequest  
        .post(requestUri)  
        .headers(headers)  
        .readTimeout(getTimeout(transactionMtid))  
        .body(payload)  
    )  
  
  private val spreadUpperBound: Int = 940 // some hard code :) ~50ms of spread we are getting from inner delays  
  
  private def makePauseForSpreading(): UIO[Unit] =  
    Random.nextIntBetween(0, spreadUpperBound).flatMap(n => ZIO.logInfo(s"Sleeping $n...") *> ZIO.sleep(n.millis))  
  
  private def sendRequest(req: RequestT[Identity, Either[String, String], Any], message: Message): UIO[SttpResponse] =  
    (backend  
      .send(req) @@ timer("ads_marmot_day_proxy_timeouts", prometheusConf, message)  
      .tagged("mt_id", message.messageTypeId)  
      .trackDuration @@ payloadCounter  
      .tagged("mt_id", message.messageTypeId))  
      .foldZIO(  
        err => {  
          val res = handleFailedRequest(err)  
          ZIO.succeed(res.resp) @@ errorCounter  
            .tagged("type", res.failureTypeTag)  
            .tagged("mt_id", message.messageTypeId)  
        },  
        resp => handleNonFailedRequest(resp, message.messageTypeId),  
      )  
  
  private def handleNonFailedRequest(resp: Response[Either[String, String]], mtid: String): UIO[SttpResponse] =  
    resp.body.fold(  
      nonSuccess =>  
        ZIO.succeed(NonSucceedResponse(Response(nonSuccess, resp.code))) @@ errorCounter  
          .tagged("type", "non_200_status_code")  
          .tagged("mt_id", mtid),  
      success => ZIO.succeed(SucceedResponse(Response(success, resp.code))),  
    )  
  
  private case class FailedResponsePlaceholder(resp: FailedResponse, failureTypeTag: String)  
  private def handleFailedRequest(err: Throwable): FailedResponsePlaceholder = {  
    val msg = err match {  
      case _: ConnectException => (s"Connection exception occurred", "connection_exception")  
      case _: TimeoutException => (s"Timeout exception occurred", "timeout_exception")  
      case _: ReadException    => (s"Read exception occurred", "read_exception")  
    }  
    FailedResponsePlaceholder(FailedResponse(msg._1, Cause.fail(err)), msg._2)  
  }  
  
  private def getTimeout(transactionMtid: String) =  
    scala.concurrent.duration.Duration(  
      appConf.mtidHttpTimeoutMillis.getOrElse(transactionMtid, appConf.httpTimeoutDefaultMillis),  
      "milli",  
    )  
}  
  
object HttpRequestsServiceImpl {  
  val layer = ZLayer {  
    for {  
      prometheusConf <- ZIO.service[PrometheusConf]  
      appConf        <- ZIO.service[AppConf]  
      backend        <- HttpZioBackend.backend  
    } yield new HttpRequestsServiceImpl(appConf, prometheusConf, backend)  
  }  
}
```

tsv
```
txn  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}  
{"messageTypeId":"20","orgId":201,"messageId":"240922299963860963","unixTimestampMs":51673000,"timezoneId":"Europe/Moscow","payload":{"mcc":"5818","ATM_ID":"550888","ORG_ID":"201","TXN_ID":"240922299963860963","ACQ_BIN":"465882","country":"RU","qvc_pan":"4150****8076","TXN_DATE":"2024-09-22 21:21:13","full_pan":"4150974690778076","merchant":"NETPROTECTOR","auth_type":"online","card_type":"QVC","TRM_TXN_ID":"7240215048688041191","CUSTOMER_ID":"qiwi","TXN_COMMENT":"NETPROTECTOR>PLOTNIKOVO RU","TXN_TO_CURR":"643","RAW_TXN_DATE":"2024-02-15 21:21:13","TXN_CUR_RATE":"91.43","bank_comment":"NETPROTECTOR>PLOTNIKOVO RU","TXN_FROM_CURR":"643","TXN_TO_AMOUNT":"11.00","TXN_TO_PRV_ID":"37017","TXN_TO_ACCOUNT":"NETPROTECTOR, карта 4150****8076","CLIENT_SOFTWARE":"1pb-cards","TXN_FROM_AGT_ID":"78013519301","TXN_FROM_AMOUNT":"11.00","TXN_FROM_PRV_ID":"7","TXN_OPERATOR_ID":"52","AmountInUsDollar":"0.12","MRMT_ORIG_TXN_ID":"28815031540","AGT_TOTAL_BALANCE":"4904.85","PRE_TO_ACCOUNT_HASH":"5246209c09"}}
```

MarmotStream
```
package com.qiwi.ads.marmot_day  
  
import com.qiwi.ads.dm.protocol.Message  
import com.qiwi.ads.marmot_day.DateTimeUtils.{defineTimeToSleep, messageDateTime}  
import com.qiwi.ads.marmot_day.adts._  
import com.qiwi.ads.marmot_day.config.{AppConf, ProxyConf}  
import com.qiwi.ads.marmot_day.services.{HttpRequestsService, TransactionService, TransactionsFileService}  
import sttp.model.Uri  
import zio._  
import zio.stream._  
  
import java.time.Instant  
import scala.io.BufferedSource  
  
trait MarmotStream[T] {  
  def marmotStream: Stream[Throwable, TxnEmittingResult]  
  def marmotSink(txnEmittingResult: TxnEmittingResult): UIO[T]  
}  
  
object MarmotStream {  
  def marmotStream[T: Tag]() = ZStream.serviceWithStream[MarmotStream[T]](_.marmotStream)  
  def marmotSink[T: Tag](txnEmittingResult: TxnEmittingResult) =  
    ZIO.serviceWithZIO[MarmotStream[T]](_.marmotSink(txnEmittingResult))  
}  
  
class MarmotStreamImpl(  
  proxyConf: ProxyConf,  
  appConf: AppConf,  
  transactionService: TransactionService,  
  transactionsFileService: TransactionsFileService,  
  httpRequestService: HttpRequestsService,  
) extends MarmotStream[Unit] {  
  
  private lazy val uriToSendTransactions =  
    ZIO.attempt(Uri(proxyConf.http.scheme, proxyConf.http.hostname, proxyConf.http.port))  
  
  override def marmotStream: Stream[Throwable, TxnEmittingResult] =  
    for {  
      curInst <-  
        ZStream.fromZIO(Clock.instant <* (Clock.currentDateTime.tap(dt => ZIO.logInfo(s"Stream start dtTm: $dt"))))  
      res <-  
        fileSource(appConf.sourceFilePath)  
          .mapZIO(transactionService.mkMsgFromString)  
          .collectSome  
          .dropWhile(msg => Instant.ofEpochMilli(msg.unixTimestampMs).isBefore(curInst))  
          .mapZIOPar(proxyConf.maxConnections) { msg =>  
            sleepIfRequired(msg, appConf.maxSleepMillisBtwnTxns, curInst)  
          }  
          .mapZIOPar(proxyConf.maxConnections)(emitMessage(_))  
    } yield res  
  
  override def marmotSink(txnEmittingResult: TxnEmittingResult): UIO[Unit] =  
    txnEmittingResult.httpResp match {  
      case SucceedResponse(r) =>  
        ZIO.logDebug(s"Response code: ${r.code}, txn dtTm: ${txnEmittingResult.txnHistoricalDtTm}")  
      case NonSucceedResponse(r) =>  
        ZIO.logError(s"Get non 2xx response code: ${r.code}: ${r.statusText}")  
      case FailedResponse(err, cause) =>  
        ZIO.logErrorCause(err, cause)  
    }  
  
  private def sleepIfRequired(  
    message: Message,  
    maxMillisToSleep: Long,  
    curInst: Instant,  
  ): UIO[Message] =  
    for {  
      timeToSleep <- ZIO.succeed(defineTimeToSleep(message.unixTimestampMs, maxMillisToSleep, curInst))  
      _ <- ZIO.when(timeToSleep > 0)(ZIO.logInfo(s"Sleeping $timeToSleep...") *> ZIO.sleep(timeToSleep.millisecond))  
    } yield message  
  
  private def fileSource(file: String): Stream[Throwable, String] =  
    ZStream  
      .acquireReleaseWith(transactionsFileService.getFileAsSource(file))(releaseFile)  
      .flatMap(f => ZStream.fromIterator(f.getLines().drop(1), 256))  
      .tapError(e => ZIO.logErrorCause(e.getMessage, Cause.fail(e)))  
  
  private def releaseFile(bufferedSource: BufferedSource): UIO[Unit] =  
    ZIO.logDebug("File was closed").as(bufferedSource.close())  
  
  private def emitMessage(  
    message: Message,  
    headers: Map[String, String] = Map(),  
  ): Task[TxnEmittingResult] =  
    for {  
      uri           <- uriToSendTransactions  
      localDateTime <- Clock.localDateTime  
      httpRes       <- httpRequestService.makeHttpRequest(uri, message, headers)  
      msgDateTime = messageDateTime(message.unixTimestampMs, message.timezoneId)  
      res         = TxnEmittingResult(httpRes, msgDateTime, localDateTime)  
    } yield res  
  
}  
  
object MarmotStreamImpl {  
  val layer = ZLayer {  
    for {  
      proxyConf               <- ZIO.service[ProxyConf]  
      appConf                 <- ZIO.service[AppConf]  
      transactionService      <- ZIO.service[TransactionService]  
      transactionsFileService <- ZIO.service[TransactionsFileService]  
      httpRequestsService     <- ZIO.service[HttpRequestsService]  
    } yield new MarmotStreamImpl(proxyConf, appConf, transactionService, transactionsFileService, httpRequestsService)  
  }  
}
```

MarmotStreamSpec
```
package com.qiwi.ads.marmot_day  
  
import com.qiwi.ads.marmot_day.DateTimeUtils.{defineTimeToSleep}  
import com.qiwi.ads.marmot_day.TestData.testMessages  
import com.qiwi.ads.marmot_day.TestUtils._  
import com.qiwi.ads.marmot_day.adts.SucceedResponse  
import com.qiwi.ads.marmot_day.config.{AppConf, PrometheusConf, ProxyConf}  
import com.qiwi.ads.marmot_day.services.{HttpRequestsServiceImpl, TransactionServiceImpl, TransactionsFileServiceImpl}  
import zio._  
import zio.stream.ZSink  
import zio.test._  
  
object MarmotStreamSpec extends ZIOSpecDefault {  
  
  val testMessagesNum: Int = 2  
  
  override def spec =  
    suiteAll("MarmotStream spec") {  
  
      test("defineTimeToSleep function") {  
        for {  
          currInst <- Clock.instant  
          res <- ZIO.foreachPar(testMessages)(msg => ZIO.succeed(defineTimeToSleep(msg.unixTimestampMs, 86400000, currInst)))  
        } yield assertTrue(res == List[Long](-60000, 0, 3 * 60000, 65 * 60000, 130 * 60000))  
      } @@ TestAspect.before(TestClock.setTime(startDtTm))  
  
      test("Streaming messages handling without http") {  
        for {  
          testMessages1    <- generateEvenlyDistributedMessages(10)  
          handledMessages1 <- runTestStream(testMessages1, 5.seconds)  
          testMessages2    <- generateEvenlyDistributedMessages(5, 5)  
          handledMessages2 <- runTestStream(testMessages2, 4.seconds)  
        } yield assertTrue(handledMessages1.size == 6 && handledMessages2.size == 1)  
      } @@ TestAspect.before(TestClock.setTime(startDtTm))  
  
      test("Messages emitting generally") {  
        for {  
          marmotStream <- ZIO.service[MarmotStream[Unit]]  
          queue        <- Queue.bounded[TxnEmittingResult](testMessagesNum)  
          _            <- marmotStream.marmotStream.run(ZSink.fromQueue(queue)).fork  
          _            <- TestClock.adjust(24.hours)  
          res          <- queue.takeAll  
        } yield assertTrue(  
          res.size == testMessagesNum && res.foldLeft(true) { (acc, r) =>  
            r.httpResp.isInstanceOf[SucceedResponse] && acc  
          }  
        )  
      } @@ TestAspect.before(defineTestDataStartDate.flatMap(TestClock.setTime(_)))  
  
      test("Messages emitting timing")(  
        for {  
          marmotStream <- ZIO.service[MarmotStream[Unit]]  
          stream       <- ZIO.succeed(marmotStream.marmotStream)  
          fiber <- stream  
                     .runFold(true)((acc, curr) => (curr.txnHistoricalDtTm == curr.txnEmittingActualDtTm) && acc)  
                     .fork  
          _   <- TestClock.adjust(24.hours)  
          res <- fiber.join  
        } yield assertTrue(res)  
      ) @@ TestAspect.before(defineTestDataStartDate.flatMap(TestClock.setTime(_)))  
  
      test("Midnight crossing")(  
        for {  
          queue        <- Queue.bounded[TxnEmittingResult](testMessagesNum * 2)  
          marmotStream <- ZIO.service[MarmotStream[Unit]]  
          stream       <- ZIO.succeed(marmotStream.marmotStream)  
          _ <- (stream.run(ZSink.fromQueue(queue))).repeat(Schedule.forever).fork  
          _   <- TestClock.adjust(48.hours)  
          res <- queue.takeAll  
        } yield assertTrue(  
          res.size == testMessagesNum * 2 && res.foldLeft(true) { (acc, curr) =>  
            {  
              println(res.size == testMessagesNum * 2)  
              acc && (curr.txnEmittingActualDtTm == curr.txnHistoricalDtTm)}  
          }  
        )  
      ) @@ TestAspect.before(defineTestDataStartDate.flatMap(TestClock.setTime(_)))  
  
    }.provideShared(  
      AppConf.layer,  
      ProxyConf.layer,  
      PrometheusConf.layer,  
      TransactionServiceImpl.layer,  
      TransactionsFileServiceImpl.layer,  
      HttpRequestsServiceImpl.layer,  
      MarmotStreamImpl.layer,  
      ZioTestHttpBackend.layer,  
    ) @@ TestAspect.timeout(1.minutes)  
  
}
```

TestData
```
package com.qiwi.ads.marmot_day  
  
import com.qiwi.ads.dm.protocol.Message  
import com.qiwi.ads.marmot_day.DateTimeUtils.dtFormatSec  
import com.qiwi.ads.marmot_day.TestUtils.makeRandomString  
import io.circe.Json  
  
import java.time._  
import scala.util.Random  
  
object r extends App {  
  println(Instant.parse("2024-09-22T14:21:13Z").atZone(ZoneId.of("Europe/Moscow")).toInstant.toEpochMilli)  
  // 1728373061093 -> 2024-10-08T07:37:41.093 (UTC) -> 2024-10-08T10:37:41.093 (Msk)  
//  println(Instant.ofEpochMilli(1728373061093L))  
//  println(Instant.ofEpochMilli(1728373061093L).atZone(ZoneId.of("Europe/Moscow")))  
  
  // 1970-01-01T00:00:00Z//  println(Clock.fixed(Instant.EPOCH, ZoneId.of("UTC")).instant().toEpochMilli)  
  
//  println(Instant.ofEpochMilli(1727014873565L))  
//  println(LocalDateTime.parse("2020-01-08 10:37:41.108", dtFormatMs).atZone(ZoneId.of("Europe/Moscow")).toInstant.toEpochMilli)  
  
  val msg = Message(  
    Random.nextInt(),  
    "20",  
    makeRandomString(5),  
    1602142661108L,  
    "Europe/Moscow",  
    Map(),  
  )  
  
//  println(LocalDateTime.parse("2024-10-08T10:37:41.108"))  
  // 1602142661108 -> 2020-10-08T07:37:41.093Z -> 2020-10-08 10:37:41.108 <- 2024-10-08 10:37:41.108 <-1728373061108//  println(messageEmittingDtTm(msg.unixTimestampMs, msg.timezoneId).compareTo(LocalDateTime.parse("2024-10-08 10:37:41.108", dtFormatMs)))  
//  println(messageEmittingMs(msg.unixTimestampMs, msg.timezoneId) == 1728373061108L)  
  
//  println(messageEmittingDtTm(msg))  
//  println(messageEmittingMs(msg))  
  
//  val inst = Instant.ofEpochMilli(1728373061093L)  
//  println(inst, inst.toEpochMilli)  
//  println(inst.atZone(ZoneId.of("Europe/Moscow")))  
//  println()  
//  println(inst.atZone(ZoneId.of("UTC")))  
//  println(inst.atZone(ZoneId.of("Europe/Moscow")).toLocalTime)  
  
//  println(Clock.fixed(Instant.EPOCH, ZoneId.of("UTC")).instant())  
//  println(Clock.systemUTC().instant().atZone(ZoneId.of("Europe/Moscow")).toLocalDate)  
//  println(Clock.system(ZoneId.of("UTC")).instant())  
//  println(inst.atZone(ZoneId.of("Europe/Moscow")).toLocalTime.atDate(Clock.systemUTC().getZone))  
//  println(inst.atZone(ZoneId.of("UTC")).toInstant)  
  
//  println(Duration.ofDays(1))  
//  println(Period.ofDays(1))  
  
  val now = LocalDateTime.now()  
  println(now)  
  println(now.atZone(ZoneId.of("Europe/Moscow")).toInstant)  
  println(now.atZone(ZoneId.of("UTC")).toInstant.toEpochMilli)  
  println(Instant.ofEpochMilli(now.atZone(ZoneId.of("UTC")).toInstant.toEpochMilli))  
}  
  
object TestData {  
  
  val testMessages: List[Message] = List(  
    "1969-12-31T23:59:00Z",  
    "1970-01-01T00:00:00Z",  
    "1970-01-01T00:03:00Z",  
    "1970-01-01T01:05:00Z",  
    "1970-01-01T02:10:00Z",  
  ).map(date =>  
    Message(  
      Random.nextInt(),  
      "20",  
      makeRandomString(5),  
      Instant.parse(date).toEpochMilli,  
      "Europe/Moscow",  
      Map(),  
    )  
  )  
  
  lazy val message1 = Message(  
    201,  
    "10",  
    "2625943012",  
    1727014873565L,  
    "Europe/Moscow",  
    Map(  
      "TXN_DATE"     -> Json.fromString(LocalDateTime.now().format(dtFormatSec)),  
      "RAW_TXN_DATE" -> Json.fromString(LocalDateTime.now().format(dtFormatSec)),  
    ),  
  )  
  
  lazy val message2 = Message(  
    201,  
    "20",  
    "2626025844",  
    1727014873565L,  
    "Europe/Moscow",  
    Map("TXN_DATE" -> Json.fromString(LocalDateTime.now().format(dtFormatSec))),  
  )  
  lazy val message3 = Message(  
    201,  
    "30",  
    "2626025516",  
    1727014873565L,  
    "Europe/Moscow",  
    Map("TXN_DATE" -> Json.fromString(LocalDateTime.now().format(dtFormatSec))),  
  )  
  
  lazy val jsonMessage1 =  
    """{  
      |  "messageTypeId": "20",      |  "messageId": "240922382435853747",      |  "orgId": 201,      |  "unixTimestampMs": 1727014873000,      |  "timezoneId": "Europe/Moscow",      |  "payload": {      |    "IP": "145.255.8.90",      |    "ORG_ID": "201",      |    "TXN_ID": "240922382435853747",      |    "ga_cid": "11d2f198",      |    "TXN_DATE": "2024-09-22 17:21:13",      |    "TRM_TXN_ID": "539346201614",      |    "CUSTOMER_ID": "qiwi",      |    "TXN_TO_CURR": "643",      |    "RAW_TXN_DATE": "2024-02-15 21:21:13",      |    "TXN_CUR_RATE": "91.43",      |    "cbr_category": "qw_outcoming",      |    "chkt_account": "samoilov_d102",      |    "ACQ_CARD_MASK": "546906******2442",      |    "TXN_FROM_CURR": "643",      |    "TXN_TO_AMOUNT": "822.87",      |    "TXN_TO_PRV_ID": "42530",      |    "TXN_UA_CRC_32": "f20529a5",      |    "PRE_IP_OR_HASH": "145.255.8.90",      |    "TXN_TO_ACCOUNT": "samoilov_d102",      |    "CLIENT_SOFTWARE": "WEB v4.127.2",      |    "OAUTH_CLIENT_ID": "web-qw",      |    "TXN_FROM_AGT_ID": "78013775333",      |    "TXN_FROM_AMOUNT": "822.87",      |    "TXN_FROM_PRV_ID": "26222",      |    "TXN_OPERATOR_ID": "78013775333",      |    "AmountInUsDollar": "9.00",      |    "MRMT_ORIG_TXN_ID": "28815031547"      |  }      |}""".stripMargin  
  
  lazy val jsonMessage2 =  
    """{  
      |  "messageTypeId": "20",      |  "messageId": "26258950863-POST_AUTH",      |  "orgId": 201,      |  "unixTimestampMs": 1727014873000,      |  "timezoneId": "Europe/Moscow",      |  "payload": {      |    "TXN_ID": "26258950863",      |    "SOURCE_TXN_DATE": "2024-09-22 17:21:13",      |    "RAW_TXN_DATE": "2024-09-22 17:21:13"      |  }      |}""".stripMargin  
  
}
```

TestUtils
```
package com.qiwi.ads.marmot_day  
  
import com.qiwi.ads.dm.protocol.Message  
import com.qiwi.ads.marmot_day.DateTimeUtils.{defineTimeToSleep, dtFormatSec}  
import com.qiwi.ads.marmot_day.config.AppConf  
import com.qiwi.ads.marmot_day.services.TransactionService  
import io.circe.Json  
import zio._  
import zio.stream.{ZSink, ZStream}  
import zio.test.{TestAspect, TestClock}  
  
import java.time.{Instant, ZoneId}  
import scala.io.{BufferedSource, Source}  
import scala.util.Random  
  
object TestUtils {  
  
  val startDtTm: Instant = Instant.EPOCH  
  
  def makeRandomString(size: Int): String = Random.alphanumeric.take(size).mkString  
  
  def runTestStream(messages: List[Message], workingTime: Duration): UIO[Chunk[Message]] =  
    for {  
      queue <- Queue.bounded[Message](10)  
      _ <- ZStream  
             .fromIterable(messages)  
             .rechunk(1)  
             .mapZIO(msg => sleepIfRequired(msg, 86400000))  
             .run(ZSink.fromQueueWithShutdown(queue))  
             .fork  
      _   <- TestClock.adjust(workingTime)  
      res <- queue.takeAll  
    } yield res  
  
  def sleepIfRequired(  
    message: Message,  
    maxMillisToSleep: Long,  
  ): UIO[Message] =  
    for {  
      curInst     <- Clock.instant  
      timeToSleep <- ZIO.succeed(defineTimeToSleep(message.unixTimestampMs, maxMillisToSleep, curInst))  
      _ <- ZIO.when(timeToSleep > 0)(ZIO.logDebug(s"Sleeping $timeToSleep...") *> ZIO.sleep(timeToSleep.millisecond))  
    } yield message  
  
  def setTimeToFirstTransaction(): TestAspect[Nothing, TransactionService with AppConf, Throwable, Any] =  
    TestAspect.before(  
      for {  
        testDataStartDate <- defineTestDataStartDate  
        _                 <- TestClock.setTime(testDataStartDate)  
      } yield ()  
    )  
  
  def generateEvenlyDistributedMessages(n: Int, period: Long = 1L): UIO[List[Message]] =  
    for {  
      currDtTm <- Clock.currentDateTime  
      res <- ZIO.foreach((0 until n).toList) { i =>  
               ZIO.succeed(  
                 Message(  
                   Random.nextInt(),  
                   "20",  
                   makeRandomString(5),  
                   currDtTm.plusSeconds(i * period).toInstant.toEpochMilli,  
                   "Europe/Moscow",  
                   Map(  
                     "TXN_DATE" -> Json.fromString(currDtTm.plusSeconds(i * period).format(dtFormatSec))  
                   ),  
                 )  
               )  
             }  
    } yield res  
  
  def defineTestDataStartDate: ZIO[TransactionService with AppConf, Throwable, Instant] =  
    ZIO.service[AppConf].flatMap { conf =>  
      ZIO.acquireReleaseWith(acquireFile(conf.sourceFilePath))(releaseFile) { f =>  
        for {  
          parsedMsg <- TransactionService.mkMsgFromString(f.getLines().drop(1).next())  
          message   <- ZIO.fromOption(parsedMsg).orElseFail(new RuntimeException(s"Can't parse message"))  
        } yield Instant  
          .ofEpochMilli(message.unixTimestampMs)  
          .atZone(ZoneId.of(message.timezoneId))  
          .toLocalDateTime  
          .atZone(ZoneId.of("UTC"))  
          .toInstant  
      }  
    }  
  
  private def releaseFile(bufferedSource: BufferedSource): UIO[Unit] =  
    ZIO.logDebug("File was closed").as(bufferedSource.close())  
  
  private def acquireFile(file: String): RIO[AppConf, BufferedSource] =  
    ZIO.attempt(Source.fromFile(file)) <* ZIO.logInfo("File was opened")  
  
}
```

TransactionService
```
package com.qiwi.ads.marmot_day.services  
  
import com.qiwi.ads.dm.protocol.{DmProtocol, Message}  
import com.qiwi.ads.marmot_day.DateTimeUtils.{dtFormatSec, messageEmittingDtTm, messageEmittingMs}  
import com.qiwi.ads.marmot_day.config.AppConf  
import com.qiwi.ads.marmot_day.metrics.errorCounter  
import io.circe.Json  
import io.circe.parser.parse  
import io.circe.syntax._  
import zio._  
import zio.macros.accessible  
  
import java.time.LocalDateTime  
import java.time.format.DateTimeFormatter  
  
@accessible  
trait TransactionService {  
  def mkMsgFromString(txn: String): UIO[Option[Message]]  
}  
  
class TransactionServiceImpl(appConf: AppConf) extends TransactionService {  
  private val dtFormatMin: DateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:00")  
  private val dateFmt: DateTimeFormatter     = DateTimeFormatter.ofPattern("yyMMdd")  
  
  override def mkMsgFromString(txn: String): UIO[Option[Message]] =  
    (for {  
      currDtTm   <- Clock.currentDateTime  
      messageRaw <- ZIO.fromEither(parse(txn)).mapError(e => new RuntimeException(e))  
      message    <- ZIO.fromEither(DmProtocol.parse(messageRaw)).mapError(e => new RuntimeException(e))  
      msgEmittingDtTm = messageEmittingDtTm(message.unixTimestampMs, message.timezoneId, currDtTm.toLocalDate)  
      rndMsgId <- genMsgIdWithDatePrefix()  
      msgAttrsOriginal = getOriginalTxnAttrs(message.payload)  
      msgAttrsMutated  = getMutatedTxnAttrs(message.payload, rndMsgId, msgEmittingDtTm)  
    } yield Message(  
      orgId = message.orgId,  
      messageTypeId = message.messageTypeId,  
      messageId = addSuffixIfRequired(rndMsgId, message.messageId),  
      unixTimestampMs = messageEmittingMs(message.unixTimestampMs, message.timezoneId, currDtTm.toLocalDate),  
      timezoneId = message.timezoneId,  
      payload = msgAttrsOriginal ++ msgAttrsMutated,  
    )).foldZIO(  
      err =>  
        ZIO.logErrorCause(  
          s"Can't build message from $txn: ${err.getMessage}",  
          Cause.fail(err),  
        ) *> ZIO.none @@ errorCounter.tagged("type", "parse_transaction"),  
      tr => ZIO.some(tr),  
    )  
  
  private def genMsgIdWithDatePrefix(): Task[String] =  
    for {  
      id       <- genRndNumOfLength(appConf.randomIdLength.min, appConf.randomIdLength.max)  
      currDtTm <- Clock.currentDateTime  
    } yield s"${dateFmt.format(currDtTm)}$id"  
  
  private def genRndNumOfLength(minLength: Int, maxLength: Int): UIO[Long] =  
    ZIO.random.flatMap(_.nextLongBetween(Math.pow(10, minLength - 1).toLong, Math.pow(10, maxLength).toLong))  
  
  private def getOriginalTxnAttrs(txnAttrs: Map[String, Json]): Map[String, Json] =  
    txnAttrs.filterNot {  
      case (k, _) =>  
        k.startsWith("PRE") || msgSpecialAttrs.contains(k)  
    }  
  
  private val msgSpecialAttrs: List[String] = List(  
    "TXN_ID",  
    "TXN_DATE",  
    "RAW_TXN_DATE",  
    "TXN_PAYMENT_DATE",  
    "MESSAGE_TYPE_ID",  
    "TXN_MINUTE",  
    "FC_HOLD",  
    "FC_60",  
    "FC_100",  
    "MESSAGE_ID",  
  )  
  
  private def getMutatedTxnAttrs(  
    txnAttrs: Map[String, Json],  
    rndTxnIdWithDate: String,  
    txnDtTm: LocalDateTime,  
  ): Map[String, Json] = {  
    val origTxnIdAttr = txnAttrs.get("TXN_ID")  
    val txnIdAttrs =  
      origTxnIdAttr.fold(Map.empty[String, Json])(x =>  
        Map("TXN_ID" -> rndTxnIdWithDate.asJson, "MRMT_ORIG_TXN_ID" -> x)  
      )  
    val handledDtTmAttrs = handleDtTmAttrs(txnAttrs, txnDtTm)  
    txnIdAttrs ++ handledDtTmAttrs  
  }  
  
  private def handleDtTmAttrs(txnAttrs: Map[String, Json], txnDtTm: LocalDateTime): Map[String, Json] = {  
    val rawTxnDate = txnAttrs.get("TXN_DATE").orElse(txnAttrs.get("SOURCE_TXN_DATE")).map("RAW_TXN_DATE" -> _)  
    val secFormatAttrs =  
      List("TXN_DATE", "SOURCE_TXN_DATE", "TXN_PAYMENT_DATE").map(x =>  
        replaceDtTmAttr(txnAttrs, x, txnDtTm, dtFormatSec)  
      )  
    val minFormatAttrs =  
      List("TXN_MINUTE", "FC_HOLD", "FC_60", "FC_100").map(x => replaceDtTmAttr(txnAttrs, x, txnDtTm, dtFormatMin))  
  
    (rawTxnDate +: (secFormatAttrs ++ minFormatAttrs)).collect {  
      case pair: Some[(String, Json)] =>  
        pair.get  
    }.toMap  
  }  
  
  private def replaceDtTmAttr(  
    txnAttrs: Map[String, Json],  
    dtTmAttr: String,  
    newDtTmAttrValue: LocalDateTime,  
    formatter: DateTimeFormatter,  
  ): Option[(String, Json)] = txnAttrs.get(dtTmAttr).map(_ => dtTmAttr -> formatter.format(newDtTmAttrValue).asJson)  
  
  private def addSuffixIfRequired(genMsgId: String, origMsgId: String): String = {  
    val split = origMsgId.split('-')  
    if (split.length > 1) s"$genMsgId-${split(1)}" else genMsgId  
  }  
}  
  
object TransactionServiceImpl {  
  lazy val layer: URLayer[AppConf, TransactionServiceImpl] = ZLayer.fromZIO(  
    ZIO.serviceWith[AppConf](conf => new TransactionServiceImpl(conf))  
  )  
}
```

TransactionServiceSpec
```
package com.qiwi.ads.marmot_day.services  
  
import com.qiwi.ads.dm.protocol.Message  
import com.qiwi.ads.marmot_day.TestData  
import com.qiwi.ads.marmot_day.TestUtils.startDtTm  
import com.qiwi.ads.marmot_day.config.AppConf  
import io.circe.Json  
import zio.test.{Spec, TestAspect, TestClock, TestEnvironment, ZIOSpecDefault, assertTrue}  
import zio.{Scope, ZIO}  
  
import java.time.format.DateTimeFormatter  
import java.time.{LocalDateTime, ZoneOffset}  
  
object TransactionServiceSpec extends ZIOSpecDefault {  
  override def spec: Spec[TestEnvironment with Scope, Any] =  
    suiteAll("Transaction service specification") {  
  
      test("parsing ordinary message") {  
        for {  
          appConf <- ZIO.service[AppConf]  
          message <- TransactionService.mkMsgFromString(TestData.jsonMessage1).some  
        } yield assertTrue(  
          checkMessage(  
            message,  
            appConf,  
            "20",  
            Map(  
              "MRMT_ORIG_TXN_ID" -> "240922382435853747",  
              "TXN_DATE"         -> "1970-01-01 17:21:13",  
              "RAW_TXN_DATE"     -> "2024-09-22 17:21:13",  
              "TXN_TO_CURR"      -> "643",  
              "TXN_TO_AMOUNT"    -> "822.87",  
            ),  
          ) && !message.payload.exists(x => x._1.startsWith("PRE"))  
        )  
      }  
      test("parsing extraordinary transaction") {  
        for {  
          appConf <- ZIO.service[AppConf]  
          message <- TransactionService.mkMsgFromString(TestData.jsonMessage2).some  
        } yield assertTrue(  
          checkMessage(  
            message,  
            appConf,  
            "20",  
            Map(  
              "TXN_ID"           -> "700101512580245785",  
              "SOURCE_TXN_DATE"  -> "1970-01-01 17:21:13",  
              "RAW_TXN_DATE"     -> "2024-09-22 17:21:13",  
              "MRMT_ORIG_TXN_ID" -> "26258950863",  
            ),  
          )  
        )  
      }  
    }.provide(  
      TransactionServiceImpl.layer,  
      AppConf.layer,  
    ) @@ TestAspect.before(TestClock.setTime(startDtTm))  
  
  private def checkMessage(  
    message: Message,  
    appConf: AppConf,  
    expMsgTypeId: String,  
    expProperties: Map[String, String],  
  ): Boolean =  
    message.messageTypeId == expMsgTypeId &&  
      checkMsgId(message.messageId, appConf) &&  
      checkMsgProps(message.payload, expProperties)  
  
  private val msgIdPrefix =  
    LocalDateTime.ofInstant(startDtTm, ZoneOffset.UTC).format(DateTimeFormatter.ofPattern("yyMMdd"))  
  
  private def checkMsgId(msgId: String, appConf: AppConf): Boolean = {  
    val randomTxnIdLength = msgId.split('-')(0).length - msgIdPrefix.length  
  
    appConf.randomIdLength.min <= randomTxnIdLength && randomTxnIdLength <= appConf.randomIdLength.max &&  
    msgId.startsWith(msgIdPrefix)  
  }  
  
  private def checkMsgProps(properties: Map[String, Json], expProperties: Map[String, String]): Boolean =  
    expProperties.forall {  
      case (k, v) =>  
        properties.get(k).forall(_.asString.forall(_.equals(v)))  
    }  
  
}
```