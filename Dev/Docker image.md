1. Добавить зависимость в `project/plugins.sbt`
```
resolvers += "Typesafe repository" at "https://repo.typesafe.com/typesafe/releases/"  
addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "1.7.2")
```
2. Добавить плагины и класс запуска в `build.sbt`
```
enablePlugins(JavaAppPackaging)  
enablePlugins(DockerPlugin)

mainClass := Some("ru.arlen.phonebook.Main")
```
3. Создать образ (версия будет `0.1.0-SNAPSHOT` или та что указана в `build.sbt` в параметре `version := "1"`)
```
sbt docker:publishLocal
```
4. Запустите образ (параметр `-p<exteranl_port>:<inner_port>` пробрасывает порты)
```
docker run --rm -p8080:8080 hello-world:0.1-SNAPSHOT
```

Примечание:
1. В веб приложении чтобы обрабатывать запросы из сети host следует указывать `"0.0.0.0"`, иначе обрабатываться будут только запросы с localhost.
2. Dockerfile можно найти по пути `target/docker/Dockerfile`
3. По умолчанию Dockerfile создается на основе образа JDK, размер образа нашего приложения можно уменьшить указав JRE `build.sbt`:
	`dockerBaseImage := "openjdk:jre"`
4. Еще более легковесный вариант образа можно создать на основе Alpine:
	`dockerBaseImage := “openjdk:jre-alpine”`
	но чтобы такой образ был рабочим следует добавить плагин
	`enablePlugins(AshScriptPlugin)` это вызвано тем, что генерируемый Dockerfile использует **bash** а Alpine поставляется с Almquist Shell (**ash**) более урезанная оболочка чем **bash** и для генерации скрипта совместимого с **ash** используется данный плагин