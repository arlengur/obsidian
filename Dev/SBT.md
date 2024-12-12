sbt --version // вывести версию
sbt // собрать проект (нужен файл build.sbt)

project folders:
```
build.sbt
project
	build.properties
target
src/main/scala
src/test/scala
```

commands:
```
runMain <path.to.Main>
~compile // continues compile
test // runs all the tests
test:testOnly <path.to.Test> or \*Test // run single test
project <pr name> // switch to current progect
```

~/.sbt/repositories
```
[repositories]
local
Maven-Central: https://maven.osmp.ru/nexus/content/repositories/central
QIWI-Public-Repo: https://maven.osmp.ru/nexus/content/groups/public
QIWI-Releases-Repo: https://maven.osmp.ru/nexus/content/repositories/releases
QIWI-Snapshots-Repo: https://maven.osmp.ru/nexus/content/repositories/snapshots
```

build.sbt
```
scalaVersion := "2.13.8"
version := "1.0" // project version
name := "pr name"
organization := "ru.arlen"
libraryDependencies += "com.lihaoyi" %% "fansi" % "0.4.0" // colored string
// or libraryDependencies += "com.lihaoyi" % "fansi_2.13" % "0.4.0" in case of single '%'
// or libraryDependencies ++= Seq("com.lihaoyi" % "fansi_2.13" % "0.4.0") // in case miltiple libraries
// libraryDependencies += "org.scalatest" %% "scalatest" % "3.2.13" % Test // available only within test dir
```

multi project build.sbt
```
ThisBuild / scalaVersion := "2.13.8" // значение применяется ко всем модулям
ThisBuild / version := "1.0" // project version
ThisBuild / name := "pr name"
ThisBuild / organization := "ru.arlen"

lazy val core = (project in file("core"))
// or lazy val core = (project in file("core")).settings(...)
lazy val server = (project in file("server"))
// or lazy val server = (project in file("server")).dependsOn(core)

lazy val root = (project in file(".").aggregate(core, server))
```

Можно вынести все константы в файл project/Dependencies.scala:
```
object Dependencies {  
lazy val scalaTest = "org.scalatest" %% "scalatest" % "3.2.2"  
}
```

и использовать их в build.sbt:
```
libraryDependencies += scalaTest % Test
```

Подключить плагин project/plugins.sbt:
```
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "1.2.0") // создает jar
```

и теперь его можно использовать в build.sbt:
```
lazy val core = (project in file("core")).settings(
	assembly / mainClass := Some("ru.arlen.CoreApp")
)
```

jar находится в target/scala-2.13/core-assembly-1.0.jar
чтобы его запустить можно выполнить команду:
```
java -jar core-assembly-1.0.jar
```

Чтобы плагин был доступен для всех проектов scala plugins.sbt нужно положить в ./sbt/1.0/plugins/

Можно добавить свои репозитории для библиотек:
resolvers += "Repository Manager" at "https://artifacts.wavesenterprise.com/repository/we-releases"
// or resolvers ++= Seq("Repository Manager" at "https://artifacts.wavesenterprise.com/repository/we-releases")

## Run test
```
sbt "testOnly *KafkaConsumerComponentTest"
```
## Custom task
Создать CustomTask.scala:
```
object CustomTask {
	def print() {
		println("SBT custom task")
	}
}
```

Добавить таск в build.sbt:
```
lazy val printTask = taskKey[Unit]("My printer") // регистрация
printTask := { CustomTask.print }// связывание кода с task
```

Запустить:
```
sbt printTask
```

Создать UuidTask.scala:
```
import java.util.UUID

object UuidTask {
	def str(): String = {
		UUID.randomUUID.toString
	}
}
```

Добавить таск в build.sbt:
```
lazy val printTask = taskKey[Unit]("My printer") // регистрация
printTask := {
	val uuid = strTask.value
	println(s"uuid=$uuid")
	CustomTask.print
}
lazy val strTask = taskKey[String]("My string")
strTask := { UuidTask.str }
```

Запустить:
```
sbt pTask
```

## Custom setting
```
lazy val strSetting = settingKey[String]("My string setting")
strSetting := { UuidTask.str }
```

Setting вычисляется один раз а Task при каждом вызове

## Command aliases
Добавить в build.sbt:
```
addCommandAlias("alias", "compile;test;assembly")
```

Чтобы скомпилировать код для разных версий scala добавить в build.sbt:
```
val scala212 = "2.12.16"
val scala213 = "2.13.8"
...
lazy val core = (project in file("core")).settings(
	crossScalaVersions := List(scala212, scala213)
)
```

и выполнить команду: +compile

