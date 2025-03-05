IP v6 `2001:0000:130F:0000:0000:09C0:876A:130B`
IP v4 `10.201.129.160`

```scala
  def getIpPref(ip: String): String =
    InetAddress.getByName(ip.split(",").head) match {
      case address: Inet4Address =>
        address.getAddress
          .take(2)
          .map(_ & 0xff)
          .mkString(".")
      case address: Inet6Address =>
        address.getAddress
          .take(6)
          .map(String.format("%02x", _))
          .grouped(2)
          .map(_.mkString)
          .mkString(":")
      case _ => ""
    }
```

