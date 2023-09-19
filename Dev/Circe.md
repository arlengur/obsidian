# Base
- Decoder - функция преобразования из Json в пользовательский класс
- Encoder - функция преобразования объекта пользовательского класса в Json
- DecodingFailure & ParsingFailure - основные ошибки
- Cursor - тип для обхода Json структур
# Auto decoder
```scala
import cats.effect.{IO, IOApp}  
import io.circe.generic.auto.exportDecoder  
import io.circe.parser.parse  
  
object CirceJsonEx extends IOApp.Simple {  
	case class User(name: String, email: Option[String])  
	  
	val example = """{"name" : "arlen", "email": "satovritti@gmail.com"}"""  
	  
	def run: IO[Unit] = IO.println {  
		for {  
			json <- parse(example)  
			user <- json.as[User]  
		} yield user  
	}  
}
```

# Semiauto decoder
Вместо подключения автоматического декодера 
`import io.circe.generic.auto.exportDecoder ` 
можно использовать полуавтоматический декодер
```scala
implicit val decoderUser: Decoder[User] = deriveDecoder
```

# Custom decoder
Вместо подключения автоматического декодера 
`import io.circe.generic.auto.exportDecoder ` 
можно написать свой
```scala
implicit val decoderUser: Decoder[User] = Decoder.instance(  
	cur =>  
		for {  
			name <- cur.downField("name").as[String]  
			email <- cur.downField("email").as[Option[String]]  
		} yield User(name, email)  
)
```