  
# Возможные маршруты  
  
`GET /greet  `
Пример: http://localhost:9000/greet?name=Arlen&age=37  
  
`GET /greet1/:name/:age  `
Пример: http://localhost:9000/greet1/Arlen/37  
  
`GET /greet2/$name<[A-z]{5}>/:age  `
Пример: http://localhost:9000/greet2/Arlen/37  
  
# Cross Site Request Forgery (CSRF)  
Команда `+ nocsrf` в файле routes отключает security в Play  
  
Или нужно добавить в html файл  
`@()(implicit request: RequestHeader)`  
а в форме:  
```scala  
<form ...>  
  @helper.CSRF.formField  
```  
а в метод (из контроллера), который открывает эту форму:  
```scala  
def ... = Action { implicit request =>  
```  
  
# Взаимодействие с сессиями  
Передать параметр в сессию:
`Redirect(...).withSession("param" -> "value")`

Очистить сессию:
`Redirect(...).withNewSession`

# Flash
Можно сделать интерфейс более отзывчивым при помощи всплывающих сообщений
На всех страницах добавить `(implicit flash: Flash)`

А на странице main сделать вывод (например в body) `@flash.get("error")  `

А в контроллере передать сообщение пользователю `Redirect(routes.HomeController.login).flashing("error" -> "wrong creds")`

# Form
Чтобы использовать формы создать
```scala
case class LoginData(username: String, password: String)

// В контроллере
import play.api.data._  
import play.api.data.Forms._  
class HomeController @Inject() (val controllerComponents: MessagesControllerComponents) extends MessagesBaseController {

val loginForm = Form(mapping("Username" -> text(3, 10), "Password" -> text(8))(LoginData.apply)(LoginData.unapply))

def validateLoginForm = Action { implicit request =>  
    loginForm.bindFromRequest.fold(  
      formWithErrors => BadRequest(views.html.login1(formWithErrors)),  
      ld =>  
        if (TaskListInMemoryModel.validateUser(ld.username, ld.password)) {  
        Redirect(routes.HomeController.test).flashing("success" -> "bingo!")  
      } else {  
        Redirect(routes.HomeController.login).flashing("error" -> "wrong creds")  
      }  
    )  
  }
...

```

На html страничке
```play
@(loginForm: Form[LoginData])(implicit request: MessagesRequestHeader, flash: Flash)

@helper.form(action = routes.HomeController.validateLoginForm) {  
    @helper.CSRF.formField  
    <h3>Create User with Play Form</h3>  
    @helper.inputText(loginForm("Username"))  
    @helper.inputPassword(loginForm("Password"))  
    <div class="form-actions">  
       <button type="submit">Login</button>  
    </div>}
```

В роутере
`POST /validateForm controllers.HomeController.validateLoginForm`

# Testing
Добавить зависимость 
`libraryDependencies += "org.scalatestplus.play" %% "scalatestplus-play" % "5.0.0" % Test,`

## Обычный тест
```scala
import db.model.TaskListInMemoryModel  
import org.scalatestplus.play.PlaySpec  
  
class TaskListInMemoryModelSpec extends PlaySpec {  
  "TaskListInMemoryModel" must {  
    "do valid login for default user" in {  
      TaskListInMemoryModel.validateUser("Arlen", "gur") mustBe true  
    }  
  }  
}
```

Это если есть такой объект
```scala
object TaskListInMemoryModel {  
  private val users = mutable.Map[String, String]("Arlen" -> "gur")  
    
  def validateUser(username: String, password: String): Boolean = {  
    users.get(username).map(_ == password).getOrElse(false)  
  }
}
```

## Тест контроллера
```scala
import org.scalatestplus.play._  
import org.scalatestplus.play.guice._  
import play.api.test._  
import play.api.test.Helpers._  

class HomeControllerSpec extends PlaySpec with GuiceOneAppPerTest with Injecting {  
  
  "HomeController GET" should {  
  
    "render the index page from a new instance of controller" in {  
      val controller = new HomeController(stubMessagesControllerComponents())  
      val home = controller.index().apply(FakeRequest(GET, "/"))  
  
      status(home) mustBe OK  
      contentType(home) mustBe Some("text/html")  
      contentAsString(home) must include ("Welcome to Play")  
    }  
  
    "render the index page from the application" in {  
      val controller = inject[HomeController]  
      val home = controller.index().apply(FakeRequest(GET, "/"))  
  
      status(home) mustBe OK  
      contentType(home) mustBe Some("text/html")  
      contentAsString(home) must include ("Welcome to Play")  
    }  
  
    "render the index page from the router" in {  
      val request = FakeRequest(GET, "/")  
      val home = route(app, request).get  
  
      status(home) mustBe OK  
      contentType(home) mustBe Some("text/html")  
      contentAsString(home) must include ("Welcome to Play")  
    }  
  }  
}
```
