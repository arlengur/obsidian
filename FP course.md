in a sense - в некотором смысле

Разница между val & def
```scala
val replicate: (Int, String) => String = ???
def replicate(n: Int, text: String): String = ???
```

вызываются одинаково `replicate(3, "Hello")`

определение val функции (ее также называют лямбда или анонимная функция так как у нее нет имени) - это обычный объект
`(n: Int, text: String) => List.fill(n)(text).mkString`

это анонимная функция эквивалентна Function2, которая является трейтом с одним нереализованным методом - SAM (simple abstract method)
```scala
new Function2[Int, String, String]{
	def apply(n: Int, text: String) = List.fill(n)(text).mkString
}
```

можно дать этой функции имя
```scala
val replicate = (n: Int, text: String) => List.fill(n)(text).mkString
val repeat = replicate // ссылаются один объект
```

Пример SAM
```scala
trait {
	def print(msg: String): Unit
}

// SAM syntax
val console: Printer = (msg: String) => println(msg)

// Standard syntax
val console: Printer = new Printer {
	def print(msg: String): Unit = println(msg)
}
```

Таким образом val функция - это просто SAM синтаксис для Function2. И скала имеет особенность того что при вызове функции apply ее можно опустить
```scala
replicate(3, "Hello")
replicate.apply(3, "Hello")
```

Def функция характерна тем что все идет вместе: имя функции, аргументы, их типы, тип результата и тело функции. В то время как в val функции имена аргументов определены в теле функции. При вызове def функции мы можем указать мена аргументов и передать их в произвольном порядке. Def функции не являются данными то есть они не являются значениями в отличие от val функций, поэтому мы не можем передать их как параметр или присвоить переменой. Но мы можем передать def функцию как параметр если используем "Eta расширение"
```scala
def replicate ...
List(replicate _)
val replicateVal1 = replicate _
val replicateVal2: (Int, String) => String = replicate
```
такая нотация говорит компилятору преобразовать def функцию в val функцию (в скала 3 такое преобразование происходит без `_`)

HKT - функция высшего порядка, принимает другую функцию в качестве аргумента

Пример вернуть цифры из переданной строки
```scala
def selectDigits(text: String): String = text.filter(_.isDigit) // placeholder syntax
```

Testing
нужно подключить библиотеки
`"org.scalatest" %% "scalatest"` для юнит тестов
`"org.scalatestplus" %% "scalacheck-1-17"` для тестов генерирующих входные параметры
```scala
class ValueFunctionExercisesTest extends AnyFunSuite with ScalaCheckDrivenPropertyChecks {
  
  test("selectDigits examples") {
    // unit test
    assert(selectDigits("hello4world-80") == "480")
    assert(selectDigits("welcome") == "")
  }
  
  test("selectDigits length is smaller") {
    // property based test
    forAll { (text: String) =>
      assert(selectDigits(text).length <= text.length)
    }
  }
  
  test("selectDigits only returns number") {  
    forAll { (text: String) =>  
      assert(selectDigits(text).forall(_.isDigit))  
    }  
  }
}
```

Пример вернуть `*` вместо символов переданной строки
```scala
def secret(text: String): String = text.map(_ => '*')
```

Idempotence - множественный вызов функции эквивалентен одному ее вызову

Пример теста на Idempotence
```scala
  test("secret PBT") {
    forAll { (text: String) =>
      val once = secret(text)
      val twice = secret(secret(text))
      assert(once == twice)
    }
  }
```

```scala
  // 1c. Implement `isValidUsernameCharacter` which checks if a character is suitable for a username.
  // We accept:
  // - lower and upper case letters
  // - digits
  // - special characters: '-' and '_'
  // For example, isValidUsernameCharacter('3') == true
  //              isValidUsernameCharacter('a') == true
  // but          isValidUsernameCharacter('^') == false
  // Note: You might find some useful helper methods on `char`.
  def isValidUsernameCharacter(char: Char): Boolean =
    char.isLetterOrDigit || char == '-' || char == '_'
```

```scala
  // 1d. Implement `isValidUsername` which checks that all the characters in a String are valid
  // such as isValidUsername("john-doe") == true
  // but     isValidUsername("*john*") == false
  // Note: Try to use `isValidUsernameCharacter` and a higher-order function from the String API.
  def isValidUsername(username: String): Boolean =
    username.forall(isValidUsernameCharacter)
```

```scala
  test("isValidUsername") {
    forAll { (username: String) =>
      assert(isValidUsername(username) == isValidUsername(username.reverse))
    }
  }
```

Int.MinValue.abs возвращает отрицательное число

```scala
  case class Point(x: Int, y: Int, z: Int) {
    // 2a. Implement `isPositive` which returns true if `x`, `y` and `z` are all greater or equal to 0, false otherwise
    // such as Point(2, 4,9).isPositive == true
    //         Point(0, 0,0).isPositive == true
    // but     Point(0,-2,1).isPositive == false
    // Note: `isPositive` is a function defined within `Point` class, so `isPositive` has access to `x`, `y` and `z`.
    def isPositive: Boolean =
      x >= 0 && y >= 0 && z >= 0
      // forAll(_ >= 0)

    // 2b. Implement `isEven` which returns true if `x`, `y` and `z` are all even numbers, false otherwise
    // such as Point(2, 4, 8).isEven == true
    //         Point(0,-8,-2).isEven == true
    // but     Point(3,-2, 0).isEven == false
    // Note: You can use `% 2` to check if a number is odd or even,
    // e.g. 8 % 2 == 0 but 7 % 2 == 1
    def isEven: Boolean =
      x % 2 == 0 && y % 2 == 0 && z % 2 == 0
      // forAll(_ % 2 == 0)

    // 2c. Both `isPositive` and `isEven` check that a predicate holds for `x`, `y` and `z`.
    // Let's try to capture this pattern with a higher order function like `forAll`
    // such as Point(1,1,1).forAll(_ == 1) == true
    // but     Point(1,2,5).forAll(_ == 1) == false
    // Then, re-implement `isPositive` and `isEven` using `forAll`
    def forAll(predicate: Int => Boolean): Boolean =
      predicate(x) && predicate(y) && predicate(z)
  }
```

```scala
  test("Point isPositive") {
    forAll { (x: Int, y: Int, z: Int) =>
      assert(Point(x.max(0), y.max(0), z.max(0)).isPositive)
    }
  }

  test("Point forAll") {
    // Oracle техника сравнить результат нового метода с аналогичным из стандартной библиотеки
    forAll { (x: Int, y: Int, z: Int, predicate: Int => Boolean) =>
      assert(Point(x, y, z).forAll(predicate) == List(x, y, z).forall(predicate))
    }
  }
```

- whatsoever - вообще
- crucial - важный
- in a sense - в некотором смысле, по сути
- to drive the point home - чтобы довести дело до конца
- as a side note - в качестве примечания
- mystery - тайна
- tooling - инструментарий
- drastically - значительно
- intimidating - пугающий
- leak - утечка
- nitty-gritty - мельчайшие подробности
- fast track - ускорить
- fallback option - запасной вариант
- it's a bit of a pain - это немного больно
- I have a bit of a experience - у меня есть небольшой опыт
- a number of reasons - ряд причин
- as such - как таковой
- truncate - обрезать
- endless supply - бесконечный запас
- anecdote - интересная история
- argue - спорить
- dispatch - отправлять