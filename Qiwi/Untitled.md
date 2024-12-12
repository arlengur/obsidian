```
package com.qiwi.ads.aggregates.repository.impl  
  
import com.qiwi.ads.aggregates.model._  
import com.qiwi.ads.aggregates.model.internal.{AggregateDescr, Reader, Writer}  
import com.qiwi.ads.aggregates.model.time.SimpleDuration.fromDuration  
import com.qiwi.ads.aggregates.model.time.SimpleDurationAtPast  
import com.qiwi.ads.aggregates.repository.impl.scheme.QiwiPredicates._  
import com.qiwi.ads.dm.protocol.QwPayloadScheme.{TXN_FROM_AGT_ID, TXN_TO_AGT_ID}  
  
import java.time.ZoneId  
import scala.concurrent.duration.DurationInt  
  
private[impl] object NewAggregates {  
  def getAggregates: List[AggregateDescr] =  
    List(  
      qwCntTxnFromQwToSamePrv60dGap1h,  
      qwCntTopupFromNBKToQw30d,  
      `qwCntTopup10k~50kFromNBKToQw30d`,  
      `qwCntTopup50k~200kFromNBKToQw30d`,  
      `qwCntTopup20k~200kFromNBKToQw30d`,  
      `qwCntTopup50k~100kFromNBKToQw30d`,  
      qwCntSameIPFromQwLast60dGap1h,  
      qwCntSameUdidFromQw60dGap1h,  
    ).map(_.apply(201))  
  
  val qwCntTxnFromQwToSamePrv60dGap1h = (orgId: Int) =>  
    AggregateDescr(  
      aggregateId = "qwCntTxnFromQwToSamePrv60dGap1h",  
      orgId = orgId,  
      messageTypeId = "20",  
      zoneId = ZoneId.of("Europe/Moscow"),  
      valueExtractor = CounterTransformer,  
      partitionOptions = PartitionOptions(PartitionGranularity.Day, CompactionOptions.AdjustToHours),  
      writer = Writer(  
        writeCriteria = List(  
          PayloadFilter(nonTransit),  
          PayloadFilter(prvToQwNotConvert),  
        ),  
        storageDepth = 120.days,  
        writeKey = txnFromAgtIdAndTxnToPrvId,  
      ),  
      readers = List(  
        Reader(  
          "qwCntTxnFromQwToSamePrv60dGap1h",  
          SimpleDurationAtPast(60.days, 1.hours),  
          txnFromAgtIdAndTxnToPrvId,  
        )  
      ),  
    )  
  
  val qwCntTopupFromNBKToQw30d = (orgId: Int) =>  
    AggregateDescr(  
      aggregateId = "qwCntTopupFromNBKToQw30d",  
      orgId = orgId,  
      messageTypeId = "20",  
      zoneId = ZoneId.of("Europe/Moscow"),  
      valueExtractor = CounterTransformer,  
      partitionOptions = PartitionOptions(PartitionGranularity.Day, CompactionOptions.NoCompaction),  
      writer = Writer(  
        writeCriteria = List(  
          PayloadFilter(prvFromNBK),  
          PayloadFilter(agtFromNbk),  
          PayloadFilter(prvToQw),  
        ),  
        storageDepth = 60.days,  
        writeKey = TXN_TO_AGT_ID,  
      ),  
      readers = List(  
        Reader(  
          "qwCntTopupFromNBKToQw30d",  
          30.days,  
          TXN_FROM_AGT_ID,  
        )  
      ),  
    )  
  
  val `qwCntTopup10k~50kFromNBKToQw30d` = (orgId: Int) =>  
    AggregateDescr(  
      aggregateId = "qwCntTopup10k~50kFromNBKToQw30d",  
      orgId = orgId,  
      messageTypeId = "20",  
      zoneId = ZoneId.of("Europe/Moscow"),  
      valueExtractor = CounterTransformer,  
      partitionOptions = PartitionOptions(PartitionGranularity.Day, CompactionOptions.NoCompaction),  
      writer = Writer(  
        writeCriteria = List(  
          PayloadFilter(prvFromNBK),  
          PayloadFilter(agtFromNbk),  
          PayloadFilter(prvToQw),  
          PayloadFilter(fromNbkMin10k),  
          PayloadFilter(fromNbkMax50k),  
        ),  
        storageDepth = 60.days,  
        writeKey = TXN_TO_AGT_ID,  
      ),  
      readers = List(  
        Reader(  
          "qwCntTopup10k~50kFromNBKToQw30d",  
          30.days,  
          TXN_FROM_AGT_ID,  
        )  
      ),  
    )  
  
  val `qwCntTopup50k~200kFromNBKToQw30d` = (orgId: Int) =>  
    AggregateDescr(  
      aggregateId = "qwCntTopup50k~200kFromNBKToQw30d",  
      orgId = orgId,  
      messageTypeId = "20",  
      zoneId = ZoneId.of("Europe/Moscow"),  
      valueExtractor = CounterTransformer,  
      partitionOptions = PartitionOptions(PartitionGranularity.Day, CompactionOptions.NoCompaction),  
      writer = Writer(  
        writeCriteria = List(  
          PayloadFilter(prvFromNBK),  
          PayloadFilter(agtFromNbk),  
          PayloadFilter(prvToQw),  
          PayloadFilter(fromNbkMin50k),  
          PayloadFilter(fromNbkMax200k),  
        ),  
        storageDepth = 60.days,  
        writeKey = TXN_TO_AGT_ID,  
      ),  
      readers = List(  
        Reader(  
          "qwCntTopup50k~200kFromNBKToQw30d",  
          30.days,  
          TXN_FROM_AGT_ID,  
        )  
      ),  
    )  
  
  val `qwCntTopup20k~200kFromNBKToQw30d` = (orgId: Int) =>  
    AggregateDescr(  
      aggregateId = "qwCntTopup20k~200kFromNBKToQw30d",  
      orgId = orgId,  
      messageTypeId = "20",  
      zoneId = ZoneId.of("Europe/Moscow"),  
      valueExtractor = CounterTransformer,  
      partitionOptions = PartitionOptions(PartitionGranularity.Day, CompactionOptions.NoCompaction),  
      writer = Writer(  
        writeCriteria = List(  
          PayloadFilter(prvFromNBK),  
          PayloadFilter(agtFromNbk),  
          PayloadFilter(prvToQw),  
          PayloadFilter(fromNbkMin20k),  
          PayloadFilter(fromNbkMax200k),  
        ),  
        storageDepth = 60.days,  
        writeKey = TXN_TO_AGT_ID,  
      ),  
      readers = List(  
        Reader(  
          "qwCntTopup20k~200kFromNBKToQw30d",  
          30.days,  
          TXN_FROM_AGT_ID,  
        )  
      ),  
    )  
  
  val `qwCntTopup50k~100kFromNBKToQw30d` = (orgId: Int) =>  
    AggregateDescr(  
      aggregateId = "qwCntTopup50k~100kFromNBKToQw30d",  
      orgId = orgId,  
      messageTypeId = "20",  
      zoneId = ZoneId.of("Europe/Moscow"),  
      valueExtractor = CounterTransformer,  
      partitionOptions = PartitionOptions(PartitionGranularity.Day, CompactionOptions.NoCompaction),  
      writer = Writer(  
        writeCriteria = List(  
          PayloadFilter(prvFromNBK),  
          PayloadFilter(agtFromNbk),  
          PayloadFilter(prvToQw),  
          PayloadFilter(fromNbkMin50k),  
          PayloadFilter(fromNbkMax100k),  
        ),  
        storageDepth = 60.days,  
        writeKey = TXN_TO_AGT_ID,  
      ),  
      readers = List(  
        Reader(  
          "qwCntTopup50k~100kFromNBKToQw30d",  
          30.days,  
          TXN_FROM_AGT_ID,  
        )  
      ),  
    )  
  
  val qwCntSameIPFromQwLast60dGap1h = (orgId: Int) =>  
    AggregateDescr(  
      aggregateId = "qwCntSameIPFromQwLast60dGap1h",  
      orgId = orgId,  
      messageTypeId = "20",  
      zoneId = ZoneId.of("Europe/Moscow"),  
      valueExtractor = CounterTransformer,  
      partitionOptions = PartitionOptions(PartitionGranularity.Day, CompactionOptions.AdjustToHours),  
      writer = Writer(  
        writeCriteria = List(  
          PayloadFilter(nonLocalIp),  
        ),  
        storageDepth = 120.days,  
        writeKey = tnxFromAgtIdAndIp,  
      ),  
      readers = List(  
        Reader(  
          "qwCntSameIPFromQwLast60dGap1h",  
          SimpleDurationAtPast(60.days, 1.hours),  
          tnxFromAgtIdAndIp,  
        )  
      ),  
    )  
  
  val qwCntSameUdidFromQw60dGap1h = (orgId: Int) =>  
    AggregateDescr(  
      aggregateId = "qwCntSameUdidFromQw60dGap1h",  
      orgId = orgId,  
      messageTypeId = "20",  
      zoneId = ZoneId.of("Europe/Moscow"),  
      valueExtractor = CounterTransformer,  
      partitionOptions = PartitionOptions(PartitionGranularity.Day, CompactionOptions.AdjustToHours),  
      writer = Writer(  
        writeCriteria = List(  
          PayloadFilter(nonTransit),  
          PayloadFilter(prvToQwNotConvert),  
          PayloadFilter(fromQwMin10m),  
          PayloadFilter(nonEmptyImei),  
        ),  
        storageDepth = 120.days,  
        writeKey = TXN_FROM_AGT_ID,  
      ),  
      readers = List(  
        Reader(  
          "qwCntSameUdidFromQw60dGap1h",  
          SimpleDurationAtPast(60.days, 1.hours),  
          TXN_FROM_AGT_ID,  
        )  
      ),  
    )  
}
```

```
package com.qiwi.ads.aggregates.repository.impl.scheme  
  
import com.qiwi.ads.aggregates.model.{FilterCriteria, VerdictFilter}  
import com.qiwi.ads.dm.protocol.Message  
import com.qiwi.ads.dm.protocol.QwPayloadScheme._  
  
import java.time.temporal.ChronoField  
import java.time.LocalDateTime  
import java.time.format.DateTimeFormatter  
  
object QiwiPredicates {  
  
  val formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")  
  
  val toMobile       = TXN_TO_PRV_ID(_: Message).exists(QiwiProviders.mobile)  
  val prvToBookmaker = TXN_TO_PRV_ID(_: Message).exists(QiwiProviders.bookmaker)  
  val toWithdraw     = TXN_TO_PRV_ID(_: Message).exists(QiwiProviders.withdraw)  
  
  // TXN_FROM_PRV_ID  
  val prvFromAcqCharge = TXN_FROM_PRV_ID(_: Message).contains("26222")  
  val prvFromQw        = TXN_FROM_PRV_ID(_: Message).contains("7")  
  val prvFromNBK       = TXN_FROM_PRV_ID(_: Message).contains("4")  
  
  // TXN_TO_PRV_ID  
  // may be KZ (21190) and Lithuania (23413) will be added later  val toTele2Mobile     = TXN_TO_PRV_ID(_: Message).exists(Set("42"))  
  val prvToQw           = TXN_TO_PRV_ID(_: Message).contains("99")  
  val prvTo32911        = TXN_TO_PRV_ID(_: Message).contains("32911")  
  val prvToQwCharge     = TXN_TO_PRV_ID(_: Message).contains("20175")  
  val prvToQwNotConvert = TXN_TO_PRV_ID(_: Message).exists(_ != "1099")  
  
  // TXN_FROM_AGT_ID  
  val agtFromBookmaker = TXN_FROM_AGT_ID(_: Message).exists(QiwiProviders.bookmaker)  
  val fromQw           = TXN_FROM_AGT_ID(_: Message).exists(_.length > 10)  
  val fromQd           = TXN_FROM_AGT_ID(_: Message).contains("2")  
  val agtFromNbk       = TXN_FROM_AGT_ID(_: Message).contains("28017")  
  val fromQwMin10m    = TXN_FROM_AGT_ID(_: Message).map(_.toInt).exists(_ > 10_000_000)  
  
  // TXN_TO_AMOUNT  
  val fromNbkMin10k  = TXN_TO_AMOUNT(_: Message).exists(_ >= 10_000)  
  val fromNbkMin20k  = TXN_TO_AMOUNT(_: Message).exists(_ >= 20_000)  
  val fromNbkMin50k  = TXN_TO_AMOUNT(_: Message).exists(_ >= 50_000)  
  val fromNbkMax50k  = TXN_TO_AMOUNT(_: Message).exists(_ <= 50_000)  
  val fromNbkMax100k = TXN_TO_AMOUNT(_: Message).exists(_ <= 100_000)  
  val fromNbkMax200k = TXN_TO_AMOUNT(_: Message).exists(_ <= 200_000)  
  
  // TXN_IP  
  val nonLocalIp = TXN_IP(_: Message).exists(_ != "127.0.0.1")  
  
  // IMEI  
  val nonEmptyImei = IMEI(_: Message).forall(!_.isBlank)  
  
  // TXN_TO_AGT_ID  
  val toQw = TXN_TO_AGT_ID(_: Message).exists(_.length > 10)  
  
  // TXN_FROM_AGT_ID and TXN_TO_AGT_ID  
  val nonTransit = (msg: Message) => TXN_FROM_AGT_ID(msg).zip(TXN_TO_AGT_ID(msg)).exists(p => p._1 != p._2)  
  
  // TXN_FROM_AMOUNT  
  val nonZeroAmount = TXN_FROM_AMOUNT(_: Message).exists(_ > 0)  
  val amountG1      = TXN_FROM_AMOUNT(_: Message).exists(_ > 1)  
  val notAmtMult50  = TXN_FROM_AMOUNT(_: Message).exists(_ % 50 != 0)  
  val amountLE      = (amount: Int) => TXN_FROM_AMOUNT(_: Message).exists(_ <= amount)  
  val amountLT      = (amount: Int) => TXN_FROM_AMOUNT(_: Message).exists(_ < amount)  
  val amountGE      = (amount: Int) => TXN_FROM_AMOUNT(_: Message).exists(_ >= amount)  
  val amountGT      = (amount: Int) => TXN_FROM_AMOUNT(_: Message).exists(_ > amount)  
  
  val rub2Rub = (message: Message) => TXN_FROM_CURR(message).contains("643") && TXN_TO_CURR(message).contains("643")  
  
  val withToken = CLIENT_SOFTWARE(_: Message).exists { x =>  
    x.contains("APP_qiwi_wallet_api") &&  
    !x.contains("APP_qiwi_wallet_api v1.0") &&  
    !x.contains("APP_qiwi_wallet_api_flow v1.0")  
  }  
  
  val noComment = (msg: Message) => TXN_COMMENT(msg).isEmpty || TXN_COMMENT(msg).exists(_.isEmpty)  
  
  val cliSoftwareWebOrApi =  
    CLIENT_SOFTWARE(_: Message).exists(e => e.toLowerCase.startsWith(Seq("web", "app_qiwi_wallet_api")))  
  
  val acquiringCardMaskedPanStarts = (cardBin: String) =>  
    acquiring_card_masked_pan(_: Message).exists(_.startsWith(cardBin))  
  
  val fromAcc = TXN_FROM_ACCOUNT(_: Message).isDefined  
  
  val oauthCliAndroid = oauth_client_id(_: Message).exists(_.toLowerCase.contains("android-qw"))  
  
  val oauthMobileDevice = oauth_client_id(_: Message)  
    .exists(a => a.toLowerCase.contains("android-qw") || a.toLowerCase.contains("iphone-qw"))  
  
  val xmlTerminal = TRM_TYPE(_: Message).contains(6)  
  
  val sumOverX = (sum: Int) => TXN_FROM_AMOUNT(_: Message).exists(_ > sum)  
  
  val TXN_TYPE: Message => Option[String]       = _.payload.get("TXN_TYPE").flatMap(_.asString)  
  val TRM_ID: Message => Option[String]         = _.payload.get("TRM_ID").flatMap(_.asString)  
  val TXN_TO_ACCOUNT: Message => Option[String] = _.payload.get("TXN_TO_ACCOUNT").flatMap(_.asString)  
  val TXN_RECEIPT_DATE: Message => Option[LocalDateTime] =  
    _.payload.get("TXN_RECEIPT_DATE").flatMap(_.asString).map(dt => LocalDateTime.parse(dt, formatter))  
  val PRE_DENOMINATION_ATTEMPTS_TOT = (msg: Message) =>  
    msg.payload.get("PRE_DENOMINATION_ATTEMPTS_TOT").flatMap(_.asNumber).flatMap(_.toInt)  
  val PRE_DENOMINATION_UNKNOWN = (msg: Message) =>  
    msg.payload.get("PRE_DENOMINATION_UNKNOWN").flatMap(_.asNumber).flatMap(_.toInt)  
  
  val qdOrdinaryTxn = TXN_TYPE(_: Message).contains("0")  
  
  val trmIdAndTxnToPrvIdKey = (msg: Message) =>  
    TRM_ID(msg)  
      .zip(TXN_TO_PRV_ID(msg))  
      .map(fromTo => s"${fromTo._1}-${fromTo._2}")  
  
  val trmIdAndTxnToAccount = (msg: Message) =>  
    TRM_ID(msg)  
      .zip(TXN_TO_ACCOUNT(msg))  
      .map(fromTo => s"${fromTo._1}-${fromTo._2}")  
  
  val txnFromAgtIdAndTxnToPrvId = (msg: Message) =>  
    TXN_FROM_AGT_ID(msg)  
      .zip(TXN_TO_PRV_ID(msg))  
      .map(fromTo => s"${fromTo._1}-${fromTo._2}")  
  
  val tnxFromAgtIdAndIp = (msg: Message) =>  
    TXN_FROM_AGT_ID(msg)  
      .zip(TXN_IP(msg))  
      .map(fromTo => s"${fromTo._1}-${fromTo._2.take(6)}")  
  
  val trmType4_19 = (msg: Message) => TRM_TYPE(msg).contains(4) || TRM_TYPE(msg).contains(19)  
  
  val txnDateIsNight: Message => Boolean = TXN_RECEIPT_DATE(_: Message).exists { dt =>  
    val hour = dt.get(ChronoField.HOUR_OF_DAY)  
    hour >= 23 || hour <= 7  
  }  
  
  val trmTypesForQdCnt         = List(4, 19, 7, 33, 1540, 53)  
  val trmTypePredicateForQdCnt = TRM_TYPE(_: Message).exists(tpe => trmTypesForQdCnt.contains(tpe))  
  
  val verdictOKWriteCriteria: FilterCriteria = VerdictFilter("OK")  
  
  val virtualDeviceField: Message => Option[String] = _.payload.get("virtualDevice").flatMap(_.asString)  
  
  val virtualDeviceLongFilter = (msg: Message) =>  
    virtualDeviceField(msg).exists(f =>  
      Seq("vserial", "vspe", "virtual").contains(f.toLowerCase)  
    ) || virtualDeviceShortFilter(msg)  
  
  val virtualDeviceShortFilter = (msg: Message) =>  
    virtualDeviceField(msg).exists(f =>  
      f.contains("cashcode")  
        && f.toLowerCase().contains("com0com")  
        && !Seq("pciserial", "stnserial", "snx_serialport", "sbser", "oxmf", "vcp").contains(f.toLowerCase)  
    )  
  
  val notVirtualDeviceShortFilter = (msg: Message) => !virtualDeviceShortFilter(msg)  
  
  val noBanknoteInsertionAttempts = (msg: Message) => PRE_DENOMINATION_ATTEMPTS_TOT(msg).contains(0)  
  val noUnknownBanknotes          = (msg: Message) => PRE_DENOMINATION_UNKNOWN(msg).contains(0)  
  val nonZeroInsertionAttemptsOrUnknownBanknotes = (msg: Message) =>  
    PRE_DENOMINATION_UNKNOWN(msg).exists(_ > 0) || PRE_DENOMINATION_UNKNOWN(msg).exists(_ > 0)  
}
```