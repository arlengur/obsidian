Функции - объекты первого рода т.е.:
- Они могут быть определены где угодно, даже внутри других функций
- Они могут быть переданы как параметры в функцию и возвращены как результат
- Существует набор операторов чтобы составлять функции

## ФП vs ООП
ОО язык - поддерживает классы (шаблоны для создания объектов) и объекты (экземпляры классов). Есть наследование, полиморфизм.
ФП - программы состоят из чистых функций (без сайд эффектов)
ФП язык - язык для которого функция - first class citizen (функции высшего порядка)

Скала - это ООП язык тк каждое значение в Скала - это объект
Пример:
val a = 1 + 1  // инфиксная форма раскрывающаяся в 1.+(1): мы вызвали метод "+" на объекте 2 (в скала нет понятия оператора)
val b = println("Hi") // Unit

Скала - это ФП язык тк каждая функция в Скала - это значение
val inc: Int => Int = x => x + 1
inc(1)
val c = inc

Вывод:
Каждая функция - это объект

Скала путь:
Взять ООП для модульности и ФП для масштабируемости.

## Структура проекта
```
project-root/
 --project/
     plugins.sbt
 --src/
    --main/
      --resources
      --scala
      --java
    --test
      --resources
      --scala
      --java
build.sbt
```

# Helloworld
Чтобы вывести приветственное слово достаточно написать:
```scala
object Test extends App {
 println(“Hello world!”)
}
```

Чтобы считать пользовательский ввод используется метод readline:
```scala
import scala.io.StdIn.readLine
object Test extends App {
 println(“Your name:”)
 val name = readLine()
 println(“Hello, ” + name)
}
```

Переменные
val x = “value” - именованное значение вычисляется сразу
def x = “value” - функция без параметров вычисляется каждый раз когда ее вызывают
lazy val x = “value” - вычисляется при первом обращении
var x = “value” - переменная, может менять значение

val x = 1
var y = 1
потом можно так: val z = x + y

val z = x + y
можно до
def x = 1
lazy var y = 1

Задачи: считать произведение 3х чисел:
```scala
readLine().split(" ").map(_.toInt).product
или
readLine().split(" ").map(_.toInt).reduceLeft(_ * _)
или
val x1, x2, x3 = readInt()
println(x1*x2*x3)
или
List.fill(3)(StdIn.readInt).product
или
Iterator.continually(Try(readInt)).takeWhile(_.isSuccess).map(_.get).product
```

Считать построчно 2 целых числа и вычислить их разницу:
```scala
Iterator.continually(readInt()).take(2).reduce(_-_)
или
readLine().split(' ').map(_.toInt).reduce(_-_)
```

# Комментарии
```scala
// A comment!
/* Another comment */
/** A documentation comment */
```

# Массивы
 
Пример создания массива:
`val str = Array("one", "two", "three")`
 
Пример параметризованного массива:
`val str = Array[String]("one", "two", "three")`
 
Размер массива указывается в круглых скобках:
`val greetStrings = new Array[String](2)`
 
Для каждой функции можно вызвать метод foreach и передать внутрь функцию.
Example:
```scala
val str = Array("one", "two", "three")
str.foreach(arg => println(arg)) 
// или str.foreach(println)
```
 
Также можно применять цикл for.
Example:
```scala
val str = Array("one", "two", "three")
for (s <- str)
  println(s)
```
 
В этом примере s имеет тип val.
 
Когда у переменной пишем круглые скобки и передаем туда значения, то Scala трансформирует это в вызов метода apply для данной переменной.
Example:
```scala
val greetStrings = new Array[String](2)
greetStrings(0) = "Hello"
greetStrings(1) = "world!"
```
 
Этот принцип применим и к объектам, если в этих объектах действительно есть метод apply.
 
В Scala все операции это вызов методов:
 
Когда происходит присвоениепеременной для которой могут быть применены круглые скобки для одного и более агрументов, то компилятор трансофрмирует это в вызов метода update:
greetStrings(0) = "Hello" будет преобразован в greetStrings.update(0, "Hello") 

# Assert
 
Пример:
```scala
val res = Array("zero", "one", "two")
assert(res.mkString("\n") == "zero\none\ntwo")
```
 
Метод assert принимает Boolean тип и проверяет его значение, если true ничего не происходит, иначе – AssertionError.
 
# Чтение из файла
 
Пример:
```scala
import scala.io.Source
for (line <- Source.fromFile("C:/meeting.txt").getLines())
  println(line.length + " " + line)
```
 
Метод getLines возвращает `Iterator[String]`.
 
Пример форматированного вывода:
```scala
import scala.io.Source

val lines = Source.fromFile("C:/meeting.txt").getLines().toList
val longestLine = lines.reduce((a, b) => if (a.length > b.length) a else b)
val maxWidth = widthOfStr(longestLine)

for (line <- lines) {
  val numSpaces = maxWidth - widthOfStr(line)
  val padding = " " * numSpaces
  println(padding + line.length + " | " + line)
}
```

 
# Интерполяция 
Интерполяция строк: 
println(s”some string $variable”)
println(s”some string ${inst.field}”)

Raw-интерполяция:
println(”some string \”qwerty\””)
println(raw”some string “qwerty””)

f-string интерполяция: 
println(math.E)
println(f”${math.E}%.5f”)

# Операторы
Префиксные
! value

Инфиксные
a + b

Постфиксные
i ++

Создание префиксного оператора: `def unary_<symbol>`
`+, -, !, ~`

# Ленивые вычисления
Откладываем вычисления до момента когда нужен результат
- Параметры функции по имени
- Параметры функции могут: 	 
Вычисляться до вызова функции - "call by value"
Вычисляются внутри функции при обращении - "call by name"

Пример call by name:
```scala
final def getOrElse[B >: A](default: => B): B =
 if (isEmpty) default else this.get

Option(v). getOrElse(throw new RuntimeException("Err!)) // ошибка будет только если значение пустое
```

Еще пример:
```scala
def fill[A](n: Int)(elem: => A): List[A]
List.fill(10)(Random.nextInt) // каждый раз при вызове - новое случайное значение
```

# Lazy val
"Ленивые" значения - вычисляются один раз, результат сохраняется (memoization). Работает и в классах, и внутри функций

Пример:
```scala
import java.time.{Duration, Instant}

lazy val lazyCurrent = Instant.now
val current = Instant.now

Thread.sleep(1000)

Duration.between(lazyCurrent, current) 
// разница больше секунды

Превращаем call by name в lazy:
def repeat(n: Int, v : ⇒ Int) {
  lazy val cached = v // вычисляется 0 или 1 раз  
  List.fill(n)(cached)
}
```

# Ленивый список
Stream (Scala 2.12 и раньше) vs LazyList (Scala 2.13+l) - полностью ленивый

Пример:
```scala
val s = 3 #:: 2 #:: 1 #:: LazyList.empty
val ll = LazyList.fill(100000) { Random.nextInt(10) }
```

Stream
- Stream.Cons[+A](hd: A, ti: => Stream[A]) обращения к tail через lazy val. Cons ячейка вычисляет хвост при обращении и сохраняет его только до следующего звена
- Stream.Empty - аналог Nil
- Первый элемент всегда вычислен

Пример 1:
```scala
ll.map(_ * 2).take(1).toVector // вычисления будут произведены только для первого значения
```

Пример 2:
```scala
    def map(s: Stream[Int], f: Int => Int): Stream[Int] =
      if (s.isEmpty) s
      else
        f(s.head) #:: map(s.tail, f)
```

Функции которые обходят весь список форсируют его (count, foldLeft, ...)

Может быть бесконечным
```scala
    def fibs: LazyList[BigInt] =
      BigInt(0) #:: BigInt(1) #:: fibs.zip(fibs.tail).map { n => n._1 + n._2 }
    fibs.take(5).toVector
```

Минусы:
- плохо сочетаются с исключениями и побочными эффектами
- задержки - иногда тоже побочный эффект
- бесконечные последовательности можно случайно форсироватть
- overhead - память и блокировки

## Управляющие конструкции

Условные 
 if-else
 pattern matching

Циклы (все циклы возвращают Unit так как являются императивными конструкциями)
 while
 do-while
 for

## Переменные

val
- неизменяемые
- могут быть lazy

var
- изменяемые
- не могут быть lazy

# Функции

## Функция-метод
`def sum1(x: Int, y: Int): Int = x + y`
- именованные параметры
- конвертируются в функции-значения

```scala
val sum4: (Int, Int) => Int = sum1 // компилятор преобразует функцию-метод sum1 в функцию-значение
val sum5 = sum1 _
```

## Функция-значение
```scala
 val foo: Int => Int
 val sum2: (Int, Int) => Int = (a, b) => a + b
 val sum2: (Int, Int) => Int = (_: Int) + (_: Int)
 val sum2: (Int, Int) => Int = _ + _
```
- являются объектами
- можно сохранять в переменные
- можно передавать на вход другим функциям и возвращать в качестве значения

Функцию-значение sum2 (так как является объектом) можно присвоить в какое-то значение:
val sum3 = sum2

`(x: Int) => x == 9` - функциональный литерал (анонимная функция)

Анонимная функция проверяющая равенство чисел
`(x: Int, y: Int) => x == y`
 это синтаксический сахар раскрывающийся в 
 ```scala
 val equality = new Function2[Int, Int, Boolean] { 
	 def apply(x: Int, y: Int) = x == y
}
```

тип `Function2[Int,Int,Boolean]` (индекс 2 указывает на количество аргументов) обычно записывают `(Int,Int) => Boolean`
Трейт Function2 имеет метод apply, поэтому когда мы вызываем equality(10, 20) происходит вызов equality.apply(10, 20)
Так как функции являются просто объектами, то мы называем их значениями первого класса (first-class values)

Внимание 
Функцию-метод лучше использовать по умолчанию так как у него именованные аргументы и это облегчает использование.

## Partial function

способ закодировать отсутствие результата для каких-то значений

```scala
def divide(x: Int, y: Int): Int = x / y

def divide: PartialFunction[(Int, Int), Int] =
    new PartialFunction[(Int, Int), Int] {
      override def apply(v1: (Int, Int)): Int = v1._1 / v1._2
      override def isDefinedAt(x: (Int, Int)): Boolean = x._2 != 0
    }

// или
  def divide: PartialFunction[(Int, Int), Int] = {
    case x if x._2 != 0 => x._1 / x._2
  }
```

Метод collect применит функцию divide только к тем значениям которые удовлетворяют isDefinedAt:
val r5 = List((4, 2), (2, 0)).collect(divide)

# Следование типам для реализации
```scala
def partial1[A,B,C](a: A, f: (A,B) => C): B => C
def partial1[A,B,C](a: A, f: (A,B) => C): B => C = (b: B) => f(a, b)
// или так как мы уже сказали какой тип аргумента
def partial1[A,B,C](a: A, f: (A,B) => C): B => C = b => f(a, b)
```

# Currying (Каррирование)
процесс преобразования функции от нескольких аргументов в несколько функций от одного аргумента
```scala
def foo(x: Int, y: Int): Int
val foo: Int => Int => Int

def curry[A,B,C](f: (A, B) => C): A => (B => C)
def curry[A,B,C](f: (A, B) => C): A => (B => C) = (a: A) => ((b: B) => f(a, b))

// Обратное каррирование
def uncurry[A,B,C](f: A => B => C): (A, B) => C
def uncurry[A,B,C](f: A => B => C): (A, B) => C = (a: A, b: B) => f(a)(b)

```

# Композиция функций
```scala
def compose[A,B,C](f: B => C, g: A => B): A => C
def compose[A,B,C](f: B => C, g: A => B): A => C = (a: A) => f(g(a))
```

Для композиции функций скала предоставляет метод compose определенный на Function1, и andThen делающий тоже самое
```scala
g compose f
f andThen g
```

SAM Single Abstract Method

trait Printer { def print(s: String): Unit }
val p: Printer = s => println(s)

# Строки
Неизменяемая структура данных
```scala
val msg = "The absolute value of %d is %d." 
msg.format(x, abs(x)
```

# Scala type system
![](Scala_type_system.svg)

Тип - множество объектов каждый из которых обладает определенным свойством.

Unit - тип который говорит о том что метод совершает сайд эффект (метод меняет внешний контекст по отношению к своему телу). Аналог void в Java или C. Является указанием на то что метод имеет сайд-эффект.
- один единственный экземпляр
- () - литерал

Null - нужен для обратной совместимости
- один единственный экземпляр
- null - литерал
- JVM interop (для обратной совместимости)
- лучше не использовать

Nothing
- не имеет экземпляра
- Exception
- Бесконечное выполнение

# Ограничения (Bounds)

```scala
def foo[T <: A](v: T) = ??? - Т подтип А
def bar[T >: A](v: T) = ??? - Т супертип А

class Array[T](...){ // тайп конструктор
   def get(idx: Int): T
}
```

Примеры:
```scala
  lazy val file: File = ???
  lazy val source: BufferedSource = Source.fromFile(file)

  def ensureClose[S <: Closeable, R](source: S)(f: S => R): R = {
    try {
      f(source)
    } finally {
      source.close()
    }
  }

  ensureClose(source)(_.getLines().toList)
```

Лямбду нужно располагать последней среди передаваемых параметров так как фигурные скобки можно только для последней секции использовать.

# Classes & Object
- являются шаблонами для создания объектов
- классы имеют поля, методы и конструкторы
- классы могут наследовать другие классы
- экземпляры классов называются объектами

Все что внутри фигурных скобок в классе является конструктором

Объекты лениво инициализируются, то есть только в момент обращения к объекту и являются синглетонами. Также называются модулями. В скала нет аналога ключевого слова static и объект можно использовать как класс со статическими членами.

Объект-компаньон имеет доступ к приватным полям и методам своего класса.

# case class & Object
- иммутабельная структура данных
- есть объект-компаньон (который имеет apply & unapply)
- copy
- hashCode
- equals
- toString

# Trait & sealed Traits
- конкретные и абстрактные методы
- нет параметров конструктора
- не можем создавать экземпляры
- поддерживают множественное подмешивание
- можно поддерживать при создании экземпляра
- нет контекст баунда

Пример:
```scala
trait WithName {
  def name: String
}
trait WithId

// на этапе объявления
class User extends WithId {
  def id: Int
}

// при создании экземпляра
val obj = new User with WithName {
  def name: String = ???
}
```

sealed гарантирует что все наследники находятся в том же файле и это полезно при паттерн матчинге.

# Классы
Класс - шаблон для создания объектов
class Dog {
 val name = "Gray"
 def woof() = println(s"$name говорит: Гав")
}

Его тело может содержать методы, значения и определения типов. Они могут ссылаться друг на друга.

Тело класса можно оставить пустым: class Dog

Создание экземпляра 
```
val dog = new Dog
println(dog.name)
dog.woof()
```

## Параметры класса
Все параметры должны быть переданы при создании объекта.
```scala
class Dog(name: String) {
 def woof() = println(s"$name говорит: Гав")
}
val dog = new Dog("Gray")
dog.woof()
```

Чтобы параметр превратился в поле класса нужно перед его имененм поставить val, перерь на него можно явно ссылаться как на член объекта:
class Dog(val name: String) {
 def woof() = println(s"$name говорит: Гав")
}
val dog = new Dog("Gray")
dog.woof()

## Инициализация
Классы могут содержать блоки инициализации, они будут выполняться каждый раз, когда создается экземпляр:
```scala
class Dog(name: String) {
 println(s"$name родился!!!")
 def woof() = println(s"$name говорит: Гав")
}
val dog = new Dog("Gray") // Gray родился!!!
dog.woof()
```

Задание: Ваша задача - спроектировать и реализовать класс официанта. Официант умеет принимать блюдо в заказ и в конце повторять, что было заказано. Также он вежлив и представляется.
Требования
имя класса Waiter
метод для заказа блюда giveMe, принимает название блюда
метод complete, возвращает список того, что было заказано
при своем появлении официант здоровается с гостем
необходимо, чтобы была возможна следующая запись
val positions = waiter.giveMe("борщ").giveMe("хлеб").complete()

Ответ:
```scala
class Waiter(val name: String, val order: List[String] = Nil) {
  def giveMe(s: String): Waiter = new Waiter(name, order++List(s))
  def complete(): List[String] = {
    println(s"My name $name")
    order
  }
}
или
class Waiter( val name:String, val order: List[String] = List()  ) {
  if(order.isEmpty)println(s"My name $name")
  def giveMe( x:String ):Waiter = {
    new Waiter(name, x::order)
  }
  def complete():List[String] = {
    order.reverse
  }
}
или
class Waiter(val name: String, var list: List[String]) {
  println(s"My name $name")    
  def giveMe(order: String): Waiter = {
    list = list :+ order
    this
  }
  def complete(): List[String] = list
}
или
class Waiter(val name: String = "noname", val order: List[String] = List()) {
  if (order.isEmpty) println(s"My name $name")
  def giveMe(pos: String): Waiter = new Waiter(name, pos+:order)
  def complete:List[String] = order.reverse
}
или
class Waiter(val name: String = "Иван", val orders: List[String] = List.empty[String]) {
  private val ordersBuffer = mutable.Buffer[String]()
  println(s"My name $name")
  def giveMe(dish: String): Waiter = {
    ordersBuffer.append(dish)
    this
  }
  def complete = ordersBuffer.toList
}
или
class Waiter(val name: String, order: List[String]) {
  private val builder = new ListBuffer[String]()
  builder ++= order

  def giveMe(food: String): Waiter = {
    if (builder.isEmpty) {
      println(s"My name $name")
    }
    builder.append(food)
    this
  }

  def complete(): List[String] = {
    val list = builder.toList
    builder.clear()
    list
  }
}
или
class Waiter(val positions: List[String]) {
    def this(name: String, positions: List[String]) {
        this(positions)
        println(s"My name $name")
    }
    def giveMe(pos: String) = {
        new Waiter(positions :+ pos)
    }
    def complete = {
        positions
    }
}
или
class Waiter(name: String, var order: List[String]) {
  println(s"My name $name")
  def giveMe(dish: String) = { order = order :+ dish; this }
  def complete = order
}
```

# Абстрактные классы
## Трейт 
- абстрактный тип, некоторые члены которого могут быть неопределены. У него не может быть параметров, в отличие от класса (на момент scala 2.12)
```scala
trait Animal {
 def name: String
 val greeting: String = s"Hi, I'm $name"
}
```

## Абстрактный класс
Класс у которого некоторые члены которого могут быть неопределены. У него могут быть параметры.
```scala
abstract class Animal(name: String) {
 val greeting: String = s"Hi, I'm $name"
}
```

Тело класса и трейта можно оставить пустым.

## Создание экземпляров
Создать экземпляр абстрактного типа напрямую нельзя, но можно создать экземпляр "анонимного класса", доопределяющего абстрактный:
val animal = new Animal {
 def name = "Bobik"
}

## Инициализация
Абстрактные типы могут содержать блоки инициализации:
trait Animal {
 def name: String
 val greeting: String = s"Hi, I'm $name"
 println("$name is created")
}
val animal = new Animal {
 def name = "Bobik"
}

Если мы создаем экземпляр абстрактного класса, мы должны передать все параметры. Даже если все абстрактные члены определены, нам все равно нужно поставить фигурные скобки.

abstract class Animal(name: String) {
 val greeting: String = s"Hi, I'm $name"
}
val animal = new Animal("Bobik"){}

Пример:
```scala
abstract class User(name: String) {
  def friends: List[User]

  def friendsOfFriends: List[User] =
    (for {
      friend <- friends
      friend2 <- friend.friends if friend2 != this
    } yield friend2).distinct

  override def toString: String = name
}

lazy val oleg: User = new User("Oleg") {
  override def friends: List[User] = List(katya, masha)
}

lazy val katya: User = new User("Katya") {
  override def friends: List[User] = List(oleg, anton)
}

lazy val masha: User = new User("Masha") {
  override def friends: List[User] = List(katya, anton)
}

lazy val anton: User = new User("Anton") {
  override def friends: List[User] = List(katya, masha)
}

oleg.friendsOfFriends
```

Задание:
Дан трейт

trait StringProcessor {
  def process(input: String): String
}
 Напишите несколько его реализаций:
tokenDeleter - в методе process обрабатывает строку, удаляя из неё все знаки препинания.
toLowerConvertor - заменяет все прописные буквы на строчные.
shortener - если строка имеет размер больше 20 символов, он оставляет первые 20 и добавляет к ней "...".
 Реализовав их, считайте из стандартного потока ввода строку и примените к ней все написанные преобразования вызовом .process у каждого из них. Порядок: перевести в нижний регистр, удалить знаки препинания, сократить

Sample Input:

This is a Wonderful Test!
Sample Output:

this is a wonderful ...

Ответ:
```scala
    val tokenDeleter = new StringProcessor {
      override def process(input: String): String = input.replaceAll("\\p{Punct}", "")
    }

    val shortener = new StringProcessor {
      override def process(input: String): String = input match {
        case s if s.length > 20 => s.take(20) + "..."
        case s => s
      }
    }

    val toLowerConvertor = new StringProcessor {
      override def process(input: String): String = input.toLowerCase
    }

    val input = StdIn.readLine()

    val processors = List(toLowerConvertor, tokenDeleter, shortener)
      
    println(processors.foldLeft(input)((res, proc) => proc.process(res)))
или
val tokenDeleter: StringProcessor = _.replaceAll("\\p{Punct}", "")

  val shortener: StringProcessor = input =>
    if (input.length <= 20) input
    else s"${input.take(20)}..."

  val toLowerConvertor: StringProcessor = _.toLowerCase

  val transform: String => String = (toLowerConvertor.process _).andThen(tokenDeleter.process).andThen(shortener.process)

  println(transform(StdIn.readLine()))
или
val tokenDeleter = new StringProcessor {
        def process(input: String): String = input.split("[.!?\\-,]").mkString("")
    }
    
    val shortener = new StringProcessor {
        def process(input: String): String = input match {
        case input if input.size <= 20 => input
        case _ => input.take(20) + "..."}
    }
    val toLowerConvertor = new StringProcessor {
        def process(input: String): String = input.toLowerCase()
    }
    
    println(shortener.process(tokenDeleter.process(toLowerConvertor.process(readLine()))))
или шаблон
input.replaceAll("\\W+", " ")
input.replaceAll("[^a-zA-Z ]","")
```

# Объекты
Это экземпляры уникального типа, их можно рассматривать как модули - набор определений.
На них можно ссылаться по имени.

```
object Pagasus{
 val name: String = "Pegasus"
 def introduce = "I am the winged stallion"
}
```

Тип объекта можно получить через object.type:
val pegasus: Pegasus.type = Pegasus

## Инициализация
Блоки инициализации вызываются при первом упоминании объекта.
object Pagasus{
 val name: String = "Pegasus"
 def introduce = "I am the winged stallion"
 println("Pagasus is initialized")
}
val pegasus: Pegasus.type = Pegasus

## Компаньоны
Объект-компаньон, имеет то же имя, что и тип и должен быть определен в том же исходном файле.
Его можно рассматривать как набор статических определений для класса или трейта.

class Cat(val name: String, val age: Int)

object Cat{
 def apply(name: String, age: Int): Cat = new Cat(name, age)
}

## Case object
К объекту можно добавить ключевое слово case, тогда у него будет красивый toString:
case object Unikorn

Задание:
Дан список координат в трёхмерном пространстве. Вам нужно написать класс Point, который будет описывать точку в трёхмерном пространстве и объект-компаньон со следующими функциями:

apply - фабрика, принимает три координаты и возвращает экземпляр типа Point
average - принимает List[Point] и вычисляет усреднённую точку между всеми координатами, либо точку с началом осей координат, если её невозможно рассчитать
show - принимает Point и превращает её в строку, состоящую из координат разделённых через пробел
Для каждой строки будет вызвана функция apply, затем для всех точек будет вызвана функция average. На выход будет передан результат функции show, примененный к усреднённой точке.

Ответ:
```scala
class Point(val x: Double, val y: Double, val z: Double)

object Point {
  def apply(x: Double, y: Double, z: Double): Point = new Point(x, y, z)

  def average(list: List[Point]) = {
    if (list.size == 0)
      Point(0.0, 0.0, 0.0)
    else {
      val sumP = list.foldLeft((0.0, 0.0, 0.0)) {
        (x, p) => (x._1 + p.x, x._2 + p.y, x._3 + p.z)
      }
      Point(sumP._1 / list.size, sumP._2 / list.size, sumP._3 / list.size)
    }
  }

  def show(p: Point) = s"${p.x} ${p.y} ${p.z}"
}
или
case class Point(val x :Double, val y :Double, val z :Double) {
    def show: String = s"$x $y $z"
}

object Point {
    def show(p :Point) = p.show
    def average(points: List[Point]):Point = points match {
        case Nil => Point(0, 0, 0)
        case _ => {
            val length = points.length;
            val (xs, ys, zs) = points.unzip3(p => (p.x, p.y, p.z))
            Point(xs.sum/length, ys.sum/length, zs.sum/length)
        }
    }
}
или
object Point {
  def average(ps: List[Point]): Point = {
    val n = ps.length
    if (n == 0) Point(0.0, 0.0, 0.0)
    else ps.map {case Point(x, y, z) => List(x, y, z)}
           .transpose
           .map(_.sum / n) 
         match {case List(x, y, z) => Point(x, y, z)}
  }
  def show(p: Point): String = s"${p.x} ${p.y} ${p.z}"
}
или
class Point(val x: Double, val y: Double, val z: Double) {
    def +(other: Point) = new Point(this.x + other.x, this.y + other.y, this.z + other.z)
    def map(f: (Double)=>Double): Point = 
        new Point(f(this.x), f(this.y), f(this.z))
}
object Point {
    def apply(x: Double, y: Double, z: Double): Point = new Point(x, y, z)
    def average(points: List[Point]): Point = points match {
        case List() => new Point(0, 0, 0)
        case _ => {
            val len: Double = points.size.toDouble
            points.foldLeft(new Point(0, 0, 0))(_+_).map(_/len)
        }
    }
    def show(p: Point): String = s"${p.x} ${p.y} ${p.z}"
}
или
class Point( val x:Double, val y:Double, val z:Double ) {
  def plus( a: Point):Point = Point( a.x+x, a.y+y, a.z+z)
  def divide( n:Int ):Point = Point( x/n, y/n, z/n )
}

object Point {
  def apply( x: Double, y: Double, z: Double): Point = new Point(x,y,z)
  def average( list: List[Point]):Point = {
    list match  {
      case Nil => Point(0,0,0)
      case _ =>
        list.foldLeft(Point(0, 0, 0))((b: Point, a: Point) => b.plus(a))
            .divide(list.length)
    }
  }
  def show( p:Point):String = {
    List(p.x,p.y,p.z).mkString(" ");
  }
}
или
class Point(val x: Double, val y: Double, val z: Double)

case object Point {
  def apply(x: Double, y: Double, z: Double): Point = new Point(x, y, z)

  def average(points: List[Point]): Point = points match {
    case Nil => Point(0, 0, 0)
    case list: List[Point] => {
      val p = list.reduce((a, b) => Point(a.x + b.x, a.y + b.y, a.z + b.z))
      Point(p.x / list.size, p.y / list.size, p.z / list.size)
    }
  }

  def show(p: Point): String = s"${p.x} ${p.y} ${p.z}"
}
```

Задание:
Объекты-компаньоны, в которых реализован метод unapply называются extractor objects. Этот метод позволяет "разбирать" объекты, получая аргументы, из которых он был создан. Например, в кейс-классах, с которыми вы уже немного сталкивались, он реализован по умолчанию. Это позволяет записывать подобные конструкции pattern matching для кейс-классов:

case class Dog(name: String, ownerName: String)
val rex = Dog("Rex", "Josh")
val match {
 case Dog(name, ownerName) => println(s"My owner is $ownerName)
 case _ => println("I am not a dog")
}
 Ваша задача - реализовать метод unapply у объекта FacedString﻿. Считать из потока ввода строку, сделать паттерн матчинг с ней, который определит, могла ли она быть результатом некоторого вызова apply. Если получилось выделить строку, от которой она была сконструирована, вывести эту строку на экран, если нет - вывести "Could not recognize string".
 Почитать про extractor objects можно тут.

Sample Input 1:

*_*test*_*
Sample Output 1:

test
Sample Input 2:

*_*test
Sample Output 2:

Could not recognize string

Ответ:
```scala
object FacedString {
  def apply(input: String) = s"*_*$input*_*"

  def unapply(arg: String): Option[String] = {
    val reg = "\\*_\\*([a-zA-Z\\*_]+)\\*_\\*".r
    arg match {
      case reg(input) => Some(input)
      case _ => None
    }
  }
}

"*_****text*_**_*" match {
  case FacedString(q) => q
  case _ => "Could not recognize string"
}
или
object FacedString {
  private val pattern = """\*_\*(.*?)\*_\*""".r

  def apply(input: String) = s"*_*$input*_*"
  
  def unapply(arg: String): Option[String] = 
    Some(arg).collect { case pattern(s) => s }
}

object Main {
def main(args: Array[String]) {
   StdIn.readLine match {
     case FacedString(s) => println(s)
     case _ => println("Could not recognize string")
   }
}}
или
object FacedString {
  def apply(input: String) = s"*_*$input*_*"

  def unapply(arg: String): Option[String] = {
    val r = """\*_\*(.*)\*_\*""".r
    arg match {
      case r(input) => Option(input)
      case _ => None
    }
  }
}

object Main {
def main(args: Array[String]) {
    FacedString.unapply(StdIn.readLine()) match {
      case Some(s) => println(s)
      case None => println("Could not recognize string")
    }
}}
или
val pattern = "^\\*_\\*(.*)\\*_\\*$".r
или
def unapply(arg: String): Option[String] = {
    val Pattern = "(\\*_\\*)(.*)(\\*_\\*)".r
    arg match {
      case Pattern(_, text, _) => Some(text)
      case _ => Some("Could not recognize string")
    }
  }
или
object FacedString {
  private val r = "^\\*_\\*(.*)\\*_\\*$".r
  def apply(input: String) = s"*_*$input*_*"

  def unapply(arg: String): Option[String] = r.findFirstMatchIn(arg).map(_.group(1))
}

object Main {
def main(args: Array[String]) {
  FacedString.unapply(StdIn.readLine)
    .orElse(Some("Could not recognize string"))
    .foreach(println)
}}
```

## Case-классы
Специальные формы классов, можно понимать их как структуры, представляющие из себя просто набор полей.
case class Cat(name: String, age: Int)

Они также могут содержать тело с различными определениями:
case class Cat(name: String, age: Int) Х
 def greet = println("Hi! I'm $name, I'm $age years old")
}

Для них автоматически создается объект-компаньон и их сожно создавать через метод apply:
val cat = Cat("Norbert", 3)

Каждый параметр конструктора автоматически становится полем класс т.е. не нужно дописывать val:
println(cat.name)
println(cat.age)

У них автоматически генерируются:
toString
equals
hashCode

Благодаря последним двум 2 объекта класса равны когда им переданы одинаковые значения полей.

У них автоматически генерируются метод copy со списком параметров, таких же как и в конструкторе, и со значениями по умолчанию из текущего объекта:
cat.copy(name = "Барсик")

# Наследование
Любые определения типов могут содержать список базовых типов:
class A extends P1 with P2 ...
trait B extends P1 with P2 ...
object C extends P1 with P2 ...
case class D() extends P1 with P2 ...

Определения из базовых типов автоматически становятся доступными в их наследниках и их экземплярах:
trait Animal{
 def name: String
}
trait Woofing extends Animal{
 def woof() = println(s"$name говорит: гав")
}
val woofer = new Woofing{
 def name = "Барбос"
}
woofer.woof
println(woofer.name)

## Переопределение
Абстрактные определения могут быть заменены на конкретные при наследовании:
trait Animal{
 def name: String
 def greeting = s"Привет, я - $name"
}
object Pegasus extends Animal{
 val name = "Pegasus"
}
println(Pegasus.greeting)

Def без параметров можно заменить на val или lazy val.

Конкретные определения могут быть заменены на другие с помощью ключевого слова override:
trait Animal{
 def name: String
 def greeting = s"Привет, я - $name"
}
object Pegasus extends Animal{
 val name = "Pegasus"
 override def greeting = s"Я - $name, крылатый скакун"
}
println(Pegasus.greeting)

При переопределении можно сослаться на метод предка через super:
trait Animal{
 def name: String
 def greeting = s"Привет, я - $name"
}
object Pegasus extends Animal{
 val name = "Pegasus"
 override def greeting = s"{$super.greeting}, крылатый скакун"
}
println(Pegasus.greeting)

## Наследование от классов
Если в списке наследования есть класс, он должен идти первым в списке наследования:
abstract class Animal
trait Greeter
object Pegasus extends Animal with Greeter

Можно указывать только один базовый класс.

Если у базового класса есть параметры, вы должны передать их при определении объекта или класса.
abstract class Animal(name: String)
trait Greeter
object Pegasus extends Animal("Pegasus") with Greeter

Но если вы определяете trait параметры, передавать не нужно. Но при определении конкретного наследника этого trait, вам нужно будет обязательно унаследовать этот базовый класс:
abstract class Animal(name: String)
trait Greeter extends Animal
object Pegasus extends Animal("Pegasus") with Greeter

## Линеаризация трейт
Цепочки из трейт, унаследованных от одного базового трейт, упорядочивают свои реализации особенным образом:
trait class Animal
trait Greeter extends Animal
trait Mammal extends Animal
Pegasus extends Animal with Greeter with Mammal

Для того, чтобы выстроить зависимости в линию, компилятор идет по всем предкам класса, объявленным после ключевого слова extends, и назначает текущий найденный класс или трейт суперклассом всех следующих членов списка предков. Если текущий найденный класс, в свою очередь, имеет предков, к ним также применяются правила линеаризации. Полученная цепочка зависимостей становится в списке перед текущим найденным предком.
Следствия линеаризации: 
● Конструкторы классов выполняются в том порядке в котором были расставлены в 
процессе линеаризации. Последним будет выполнен конструктор создаваемого класса.
● Доступ к членам суперклассов через ключевое слово super происходит в обратном 
порядке. Т.е. super.memberName обратится к memberName ближайшего суперкласса, 
полученного в процессе линеаризации.

Пример алгебраического типа данных (каждый case class можно рассматривать как произведение типов полей, а sealed trait является логической суммой типов, которые являются наследниками):
sealed trait Node {
  def values: List[String]
  def +:(value: String): Node
  def :+(value: String): Node =
   this match {
    case leaf@Leaf(_) =>
	 Branch(leaf, Leaf(value))
	case Branch(left, right) =>
	 Branch(left, right :+ value)
   }
  def ++(node: Node): Node = Branch(this, node)
}

case class Branch(left: Node, right: Node) extends Node {
  def values = left.values ++ right.values
  def +:(value: String) = Branch(value +: left, right)
}

case class Leaf(value: String) extends Node {
  def values = List(value)
  def +:(value: String) = Branch(Leaf(value), this)
}

val tree = Branch(Branch(Leaf("one"), Leaf("two")), Leaf("three"))
tree.values

val tree2 = "zero" +: tree
tree2.values

(tree ++ tree2).values
(tree ++ tree2 :+ "four").values

Задание:
Имеется трейт "Животное", в котором реализован метод "подать голос", а также поле "звук", который животное издает.

trait Animal {
  val sound: String
  def voice: Unit = println("voice: " + sound)
}
Задание: реализуйте классы "Кошка", "Собака" и "Рыба". В последнем случае метод voice должен печатать на экран фразу "Fishes are silent!". Программа будет запускаться следующим образом:
object Main extends App {
  val animals = Seq(new Cat, new Dog, new Fish)
  animals.foreach(_.voice)
}

Sample Input:

Sample Output:
voice: Meow!
voice: Woof!
Fishes are silent!

Ответ:
```scala
class Cat extends Animal {
  override val sound = "Meow!"
}

class Dog extends Animal {
  override val sound = "Woof!"
}

class Fish extends Animal {
  override val sound = "Fishes are silent!"
  override def voice: Unit = println(sound)
}
или
case class Cat(override val sound: String = "Meow!") extends Animal
case class Dog(override val sound: String = "Woof!") extends Animal
case class Fish(override val sound: String = "Fishes are silent!") extends Animal {
  override def voice = println(sound)
}
```

Задание:
Программист Олег решил выдавать кредиты. Для этого ему нужно реализовать класс CreditBank и его метод issueCredit, который выводит на консоль слово "CREDIT". К сожалению, у Олега нет своего капитала, и он решил собирать выдаваемый кредит по кусочкам в других банках. Имеется пять банков:
trait BankA extends AbstractBank {
  override val b = 'T'
  override val d = 'R'
  override val f = 'I'
}

trait BankB extends AbstractBank {
  override val a = 'E'
  override val f = 'D'
}

trait BankC extends AbstractBank {
  override val b = 'C'
  override val d = 'D'
}

trait BankD extends AbstractBank {
  override val b = 'C'
  override val c = 'R'
  override val d = 'E'
}

trait BankE extends AbstractBank {
  override val b = 'C'
  override val a = 'I'
  override val e = 'T'
}

Все они наследуют трейт:
trait AbstractBank {
  def a: Char
  def b: Char
  def c: Char
  def d: Char
  def e: Char
  def f: Char
  def issueCredit: Unit
}

Задание: помогите Олегу собрать кредит, подмешав в нужной последовательности банки в CreditBank и реализовав метод issueCredit ("CREDIT" должен собираться из кусочков a–f).

Ответ:
class CreditBank extends AbstractBank with BankB with BankE with BankD {
  def issueCredit = println("" + b + c + d + f + a + e) //for example: a + b + c + d + e + f
}



# Модификаторы
## Final
Классы помеченные final нельзя наследовать, а члены классов помеченные final нельзя переопределять:
final class Human{
 final def powerLevel: Int = 100
}

## Sealed
Классы и трейт могут быть помечены sealed, тогда их можно наследовать только в том же файле исходного кода. Вы заранее знаете все альтернативы данного типа, это может быть полезно для pattern matching:
sealed trait Weekday
case object Monday extends Weekday
case object Tuesday extends Weekday

## Private
К членам класса помеченных private доступ возможен только из экземпляров того же класса или компаньона:
class Dog{
 private val kind: String = "dog"
}
object Dog{
 def kindOf(dog: Dog): String = dog.kind
}
(new Dog).kind // error!

## `Private[this]`
Такой член класса может быть доступен только из того же самого экземпляра, доступ из компаньона или других экземплярах этого же класса запрещен:
class Dog{
 private[this] val kind: String = "dog"
 def kindOf(dog: Dog): String = dog.kind // error!
}

## `Private[...]`
Член класса виден только из какого-то пакета или класса (полезно для вложенных классов):
package ru.course.classes
abstract class Dog{
 private[Dog] val age: Int
 private[classes] val kind: String
 private[course] val name: String
}

## Protected
Член класса виден всем потомкам этого класса. К нему можно дописывать модификаторы в квадратных скобках:
trait Animal{
 protected val kind: String
}
class Dog extends Animal{
 protected val kind = "dog"
 def kindOf(dog: Dog): String = dog.kind
}
(new Dog).kind // error!

Эти модификаторы можно дописывать к параметрвам конструктора и override:
trait Animal{
 protected val kind: String = "animal"
}
class Mammal(override protected val kind: String) extends Animal{
 def printKind = printKind(kind)
}
(new Mammal("dog")).printKind
(new Mammal("dog")).kind // error!

# Обобщенные типы
final case class NamedInt(name: String, value: Int)
final case class NamedDouble(name: String, value: Double)

final case class Named[A](name: String, value: A)
Named[Int]
Named[Double]

## Ссылка на тип
Параметр типа не обязательно передавать явно при создании экземпляра обобщенного типа, если компилятор scala может вычислить его из типов передаваемых аргументов:
final case class Named[A](name: String, value: A){
 def withName(newName: String): Named[A] = Named(newName, value)

Можно передавать его в другие обобщенные типы:
final case class Named[A](name: String, value: A){
 def toMap: Map[String, A] = Map(name -> value)
}

Класс также может иметь обобщенные методы, тогда в теле и параметрах мы можем ссылаться на параметры типа как класса, так и метода:
final case class Named[A](name: String, value: A){
 def mapValue[B](f: A => B): Named[B] = Named(name, f(value))
}

## Абстрактные типы
Трейты или классы:
trait Named[A]{
 def name: String
 def value: A
 def modify(f: A => A): Named[A]
}

При реализации типы будут проверены.
При наследовании вы можете передавать параметры типов:
trait Named[A]{
 def name: String
 def value: A
}
case class NamedList[A](name: String, value: List[A]) extends Named[List[A]]

Класс может иметь несколько параметров типа:
final case class Dict[K, V](items: List[(K, V)])

## Верхняя граница
Параметры типов могут иметь ограничение сверху т.е. тип должен быть подтипом указанного типа:
final case class Dict[I <: Item](items: List[I]){
 def dict[I <: Item](items: I*): Dict[I] = Dict(items.toList)
}
trait Item {
 def key: String
 def value: String
}

## Нижняя граница
Параметры типов могут иметь ограничение снизу т.е. тип должен быть надтипом указанного типа:
final case class Dict[I <: Item](items: List[I]){
 def +:[J >: I](items: J): Dict[J] = Dict(item :: items)
}
trait Item {
 def key: String
 def value: String
}

## Ссылки
Параметры типов могут ссылаться друг на друга в ограничениях:
case class Dict[K, V, I <: Item[K, V]](items: List[I])
trait Item[K, V] {
 def key: K
 def value: V
}

## Абстрактные типы
Тайп параметр может ссылаться сам на себя (рекурсивно-ограниченная нотификация, F-bound notification)
trait Comparable[A <: Comparable[A]]{
 def compare(x: A): Int
}

Пример:
```scala
final case class MultiMap[Key, Value](items: Map[Key, List[Value]]) {
  override def toString: String = s"MultiMap(${items.map { case (key, values) => s"$key -> ${values.mkString(",")}" }.mkString(";")})"

  def add(key: Key, value: Value): MultiMap[Key, Value] =
    MultiMap(items + (key -> (items.get(key) match {
      case Some(values) => value :: values
      case None => List(value)
    })))

  def map[B](f: Value => B): MultiMap[Key, B] =
    MultiMap(items.mapValues(_.map(f)))
}

object MultiMap {
  // * - означает что хотим передать много таких значений
  def apply[Key, Value](items: (Key, Value)*): MultiMap[Key, Value] = new MultiMap(items.groupBy(_._1).mapValues(_.map(_._2).toList))
}

val dict = MultiMap(
  "apple" -> "fruit",
  "pear" -> "fruit",
  "carrot" -> "yellow",
  "apple" -> "delicious",
  "carrot" -> "vegetable",
)
dict.add("pear", "delicious").add("melon", "yellow")
dict.map(_.toUpperCase)
dict.map(_.length)
```

Задание:
Реализуйте неизменяемый класс Pair с методом swap, возвращающим пару, где компоненты поменяны местами.

Ответ:
```scala
case class Pair[T, S](first: T, second: S) {
  def swap = new Pair[S, T](second, first)
}

val pair = Pair(123, "Oleg").swap
val pair2 = Pair("Oleg", 123.0).swap
require(pair.first ==  "Oleg")
require(pair.second == 123)
или
case class Pair[T, S](first: T, second: S) {
  def swap: Pair[S, T] = Pair(second, first)
}
```
Задание:
Исправьте определение класса Pair так, чтобы можно было создавать пары из различных типов (например, Int или String). В этой задаче элементы пары имеют одинаковый тип.

Метод smaller должен возвращать наименьшее значение из пары.

Подсказка: трэйт Ordered[A] определяет оператор сравнения, что позволяет удобно сравнивать различные элементы. Например, BigInt <: Ordered[BigInt], поэтому можно писать: BigInt(1) < BigInt(2) == true.

Дополнительная информация: подробности про View Bounds https://stackoverflow.com/questions/4465948/what-are-scala-context-and-view-bounds

Ответ:
```scala
case class Pair[T] (first: T, second: T) {
  def smaller =
    if (BigInt(first.toString) < BigInt(second.toString)) first
    else second
}

val p = Pair(BigInt("1000000000000000"),BigInt("7000000000000000"))
//val p = Pair(8,11)
p.smaller
require(p.smaller == BigInt("1000000000000000"))
или
case class Pair[T](first: T, second: T)(implicit ord: Ordering[T]) {
  def smaller: T =
    if (ord.compare(first, second) == -1) first
    else second
  def w: Int = ord.compare(first, second)
}
или
case class Pair[T <% Ordered[T]](first: T, second: T) {
  def smaller =
    if (first < second) first
    else second
}
или
case class Pair[T](first: T, second: T)(implicit im: T => Ordered[T]) {
  def smaller: T = {
    if (first < second) first
    else second
  }
}
или
case class Pair[T: Ordering](first: T, second: T) {
  def smaller =
    if (implicitly[Ordering[T]].compare(first, second) < 0) first
    else second
}
или
case class Pair[T](first: T, second: T) {
    def smaller(implicit ord: Ordering[T]) = {
        import ord.mkOrderingOps
        if (first < second) first else second
    }
}
```

# Вариантность и род

Examples:
    class FruitBasket1[+T](val fruit: T)
    class FruitBasket2[-T](
        val fruit: T
    ) // Contravariant type T occurs in covariant position in type T of value
    class FruitBasket3[T](val fruit: T)

    class FruitBasketWithVar1[+T](
        var fruit: T
    ) // Covariant type T occurs in contravariant position in type T of value fruit
    class FruitBasketWithVar2[-T](
        var fruit: T
    ) // Contravariant type T occurs in covariant position in type T of value fruit
    class FruitBasketWithVar3[T](var fruit: T)

Правило: аргументы методов - контравариантны

    trait Purchase1[+T] {
      def buy(
          fruit: T
      ) // Covariant type T occurs in contravariant position in type T of value fruit
    }
    trait Purchase2[-T] {
      def buy(fruit: T)
    }
    trait Purchase3[+T] {
      def buy[B >: T](fruit: B): Purchase3[B] // расширили допустимые типы
    }

    Правило: возвращаемые значения - ковариантны
    trait FruitStore1[+T] {
      def show(name: String): T
    }	
    trait FruitStore2[-T] {
      def show(
          name: String
      ): T // Contravariant type T occurs in covariant position in type T of value
    }
    trait FruitStore3[-T] {
      def show[B <: T](name: String, default: B): B
    }

## Ковариантность
trait Coll[+A]{
 def apply(i: Int): A
}

Тип А производит элементы типа А.
Если тип A является подтипом B, то Coll[A] является подтипом Coll[B]:
A <:< B -> Coll[A] <:< Coll[B]

Если параметр типа ковариантный, то он должен встречаться только в ковариантных позициях т.е. он встречается в типе результата или в другом ковариантном типе в типе результата.

## Контрвариантность
trait Printer[-A]{
 def print(a: A): String
}

Тип А использует или поглощает элементы типа А.
Если тип A является подтипом B, то Printer[A] является подтипом Printer[B]:
A <:< B -> Coll[B] <:< Coll[A]

Если параметр типа контрвариантный, то он должен встречаться только в контрвариантных позициях например в позициях аргумента или как аргумент ковариантного типа стоящего в аргументной позиции (def printList(as: List[A])) или как аргумент для контрвариантного типа, который в ковариантной позиции (def prefixed(s: String): Printer[A]).

## Вариантность
Функции контрвариантны по первому параметру и коварианты по второму
A => B
trait Function1[-A, +R]
Функция потребляет элементы типа А и производит элементы типа В.

В функции map А присутствует в контрвариантом типе на месте аргумента т.е. А в контр контрвариантной позиции т.о. получаем ковариантую позицию для А:
trait Coll[+A]{
 def apply(i: Int): A
 def map[B](f: A => B): Coll[B]
}

аналогично для контрвариантного типа:
trait Printer[-A]{
 def print(a: A): String
 def contrmap[B](f: B => A): Printer[B]
}

## Род
Тип типов или форма типов.
Пример простого типа род:
Int, String, List[Int]: T
Map[Int, String], Int => String: T

Пример элементов высшего рода
List, Vector: T[_]
Map, Function: T[_, _]
StateT: T[_[_], _, _]

Параметры типов могут иметь высший род т.е. мы пока не знаем набор каких элементов будет иметь тип Int:
case class IntContainer[F[_]](value: F[Int])

Можно накладывать ограничения на элементы относящиеся к высшему роду:
case class Dict[K, V, T[X] <: Seq[X]](items: T[(K, V)])

Пример:
```scala
final case class MultiMap[Key, +Value](items: Map[Key, List[Value]]) {
  override def toString: String = s"MultiMap(${items.map { case (key, values) => s"$key -> ${values.mkString(",")}" }.mkString(";")})"

  def add[B >: Value](key: Key, value: B): MultiMap[Key, B] =
    MultiMap(items + (key -> (items.get(key) match {
      case Some(values) => value :: values
      case None => List(value)
    })))

  def map[B](f: Value => B): MultiMap[Key, B] =
    MultiMap(items.mapValues(_.map(f)))
}

object MultiMap {
  // * - означает что хотим передать много таких значений
  def apply[Key, Value](items: (Key, Value)*): MultiMap[Key, Value] = new MultiMap(items.groupBy(_._1).mapValues(_.map(_._2).toList))
}

sealed trait Tag extends Product with Serializable

final case class Fact(name: String) extends Tag

final case class Personal(name: String) extends Tag

val dict = MultiMap(
  "apple" -> Fact("fruit"),
  "pear" -> Fact("fruit"),
  "carrot" -> Fact("yellow"))
dict
  .add("apple", Personal("delicious"))
  .add("carrot", Fact("vegetable"))
  .add("pear", Personal("delicious"))
  .add("melon", Fact("yellow"))
```

Задание:
Заданы тип A и его подтип B, а также функции, которые умеют распечатывать их поле value:

val printA = FunctionPrint[A]("A-value:")
val printB = FunctionPrint[B]("B-value:")

class FunctionPrint[T <: A](prefix: String) {
  def apply(t: T): Unit = println(prefix + " " + t.value)
}
Также существует важный метод methodPrint, который принимает на вход экземпляр типа B и функцию, которая умеет распечатывать его значение. methodPrint(printB, objB) компилируется без проблем, однако, иногда нужно задействовать функцию printA.
Действительно: B <: A, поэтому любая f: A => Any умеет работать и с B. Но есть один нюанс: в текущей реализации printA инвариантна к printB (не является ни родителем, ни наследником), поэтому эту функцию нельзя передать в метод methodPrint.

Задание: исправьте компиляцию кода. В конце будет вызываться:

methodPrint(printA, objB)

Подсказка: этот пример демонстрирует, почему функции в Scala контринвариантны по аргументам.

Sample Input:

A-value:
B-value:
Sample Output:

B-value: It is a B.value
A-value: It is a B.value

Ответ:
```scala
class A (val value: String)
case class B (override val value: String) extends A(value)

val objB = B("It is a B.value")
val objA = new A("It is a A.value")

def methodPrint(f: FunctionPrint[B], obj: B) = {
  f(obj)
}

class FunctionPrint[-T <: A](prefix: String) {
  def apply(t: T): Unit = println(prefix + " " + t.value)
}

object FunctionPrint {
  def apply[T <: A](prefix: String) = new FunctionPrint[T](prefix)
}

val printA = FunctionPrint[A]("A-value:")
val printB = FunctionPrint[B]("B-value:")
methodPrint(printB, objB)
methodPrint(printA, objB)
```

Задание:
Исправьте определение класса Pair , чтобы он стал ковариантным.

Метод printNames принимает на вход пары с объектами типа Person и печатает их имена. Однако нам хочется на вход этому методу передавать также и Student. Для этого нужна ковариантность пар: Pair[Student] <: Pair[Person] .

Подсказка: в определении Pair в методе replaceFirst тип T стоит в контрвариантной позиции, что мешает быть Pair ковариантным по T.

Sample Input:

Pavel Oleg
Sample Output:

1: Oliver  2: Oleg

Ответ:
```scala
class Person (val name: String)
class Student(name: String, val course: Int) extends Person(name)

case class Pair[+T](first: T, second: T) {
  def replaceFirst[B >: T](newValue: B): Pair[B] = {
    Pair(newValue, second)
  }
}

def printNames[T <: Person](pair: Pair[T]): Unit = {
  println("1: " + pair.first.name + "  2: " + pair.second.name)
}

val pair = Pair(new Student("Pavel", 1), new Student("Oleg", 5))
printNames(pair.replaceFirst(new Person("Oliver")))
```

Задание:
Укажите верное применение рода типа (wildcard parameter):
def f[M <: IndexedSeq[_]](data: M) = println(data.head)

# Псевдонимы 
Для типов можно объявлять псевдонимы:
type IntList = List[Int]

Они также могут иметь параметры:
type DenseMatrix[A] = Vector[Vector[A]]

Аргументы могут быть объявлены как ко- или контвариантными, чтобы компилятор мог проверить что мы ссылаемся на них в правильных местах:
type SparseMatrix[+A] = Map[(Int, Int), A]

Псевдонимы полностью прозрачны для компилятора, их использование эквивалентно использованию оригинального типа:
def events(xs: IntList): IntList = 
 xs.filter(_%2==0)

Они могут накладывать ограничения на свои параметры:
type Matrix[F[X] <: Iterable[X], A] = F[F[A]]

Их можно использовать чтобы создать тип правильной вариантности (желаемого рода) и передать в качестве его аргумента параметризованный обобщенный тип:
type IntMap[A] = Map[Int, A]
type DenseMatrix[A] = Matrix[IntMap, A]

## Типы-компоненты
Это типы-псевдонимы без конкретного определения, они могут различаться у разных экземпляров родительского типа:
trait Item {
 type Key
 type Value
 def key: Key
 def value: Value
}

Компилятор забывает какой конкретный тип-компонент у объекта, но тип-компонент все равно можно использовать чтобы проверить что все остальные определения в теле трейта подходят друг другу:
trait Coyoneda[A] { self =>
 type Pivot
 val start: Pivot
 val run: Pivot => A
 def value: A = run(start)
}

## Уточнение
Компилятор будет стараться запомнить как можно больше информации о типах внутри экземпляра:
trait Container { 
 type Item
 def values: List[Item]
}

val ints = new Container{
 type Item = Int
 val values = List(1,2,3)
}
ints.values // List[Int]

Это происходит благодаря структурным уточнениям типов: 
val ints: Container {type Item = Int} = new Container{
 type Item = Int
 val values = List(1,2,3)
}
val xs: List[Int] = ints.values // List[Int]

Мы можем поставить в соответствие уточненному типу параметрический псевдоним - aux-паттерн:
object Container{
 type Aux[I] = Container{type Item = I}
}

Задание:
Что будет выведено данным кодом при запуске:
type Row = List[Int]
val row: Row = Row(1, 2, 3)
println(row.mkString(","))

Ответ:
```scala
not found: value Row
т.к. Row(1, 2, 3) - не то же самое, что и List(1, 2, 3). List(1,2,3)  ~ List.apply(1,2,3).
```

Задание:
Aux - паттерн может быть использован для создания обобщённых типов без использования тайп-параметров. Данная задача - пример такого использования.

 Реализуем компактные представления массива Char и массивов Boolean (для Boolean массивы не больше 8 и 64 элементов) в памяти.

 Заведём вспомогательный трейт:

trait Vect extends Any{
  type Item
  def length: Int
  def get(index: Int): Item
  def set(index: Int, item: Item): Aux[Item]
}

object Vect {
  type Aux[I] = Vect { type Item = I }
}

 Реализуем оптимальный массив Char:
```scala
final case class StringVect(str: String) extends  AnyVal with Vect {
  type Item = Char
  def length                                             = str.length
  def get(index: Int)                                     = str.charAt(index)
  def set(index: Int, item: Char): Aux[Char] = StringVect(str.updated(index, item))
}
 Ваша задача - реализовать недостающие метода интерфейса Vect у BoolVect64 и BoolVect8:
final case class BoolVect64(values: Long) extends AnyVal with Vect {
  type Item = Boolean
  def length          = 64
  def get(index: Int) = ???
  def set(index: Int, item: Boolean) = ???
}

final case class BoolVect8(values: Byte) extends AnyVal with Vect {
  type Item = Boolean
  def length = 8
  def get(index: Int) = ???
  def set(index: Int, item: Boolean) = ???
}
```

Обратите внимание, что разряды в двоичной записи числа возрастают справа налево. Однако BoolVect8(1) должен представляться в виде List(true, false, false, false, false, false, false, false).
   
Подсказка: используйте битовые операции.
 Также реализуйте вспомогательный  метод toList, возвращающий список элементов переданного массива:
def toList(vect: Vect): List[vect.Item] = ???
 Ничего считывать или выводить не надо. Просто реализуйте недостающие методы.

Ответ:
```scala
import Vect.Aux

trait Vect extends Any {
  type Item

  def length: Int

  def get(index: Int): Item

  def set(index: Int, item: Item): Aux[Item]
}

object Vect {
  type Aux[I] = Vect {type Item = I}
}

final case class StringVect(str: String) extends Vect {
  type Item = Char

  def length = str.length

  def get(index: Int) = str.charAt(index)

  def set(index: Int, item: Char): Aux[Char] = StringVect(str.updated(index, item))
}

final case class BoolVect64(values: Long) extends Vect {
  type Item = Boolean

  def length = 64

  def get(index: Int) = (values >> index) & 1 match {
    case 1 => true
    case 0 => false
  }

  def set(index: Int, item: Boolean): Aux[Boolean] = BoolVect64(values^1L<<index)
}

final case class BoolVect8(values: Byte) extends Vect {
  type Item = Boolean

  def length = 8

  def get(index: Int) = (values >> index) & 1 match {
    case 1 => true
    case 0 => false
  }

  def set(index: Int, item: Boolean): Aux[Boolean] = 
    if (item)
      BoolVect8((values | (1 << index)).toByte)
    else
      BoolVect8((values & ~(1 << index)).toByte)
}
  def toList(vect: Vect): List[vect.Item] = (0 until vect.length).map(i => vect.get(i)).toList
или
def get(index: Int) = ((values >> index) & 1L) == 1
(for( i <- 0 until vect.length ) yield vect.get(i)).toList
или
def set(index: Int, item: Boolean): Aux[Boolean] = {
     val newValues = BigInt(values)

     if(item) {
       BoolVect64(newValues.setBit(index).toLong)
     } else {
       BoolVect64(newValues.clearBit(index).toLong)
     }
    }
def toList(vect: Vect): List[vect.Item] = {
    var list = List[vect.Item]()

    for(i <- 0 until vect.length) {
      list = list :+ vect.get(i)
    }

    list
  }
или
def toList(vect: Vect): List[vect.Item] = List.tabulate(vect.length)(n => vect.get(n))
или
def toList(vect: Vect): List[vect.Item] = (0 until vect.length).map(vect.get(_)).toList
```


# Коллекции

## Виды коллекций
- Set (т.е. без дубликатов или повторяющихся элементов)
- Seq (т.е. у каждого элемента свой индекс, например - Vector, Range, List, Array). List по умолчанию
- Map (т.е. пары ключ-значение) 


## Примеры
Examples:
```scala
val oneTwoThree = List(1, 2, 3)
val threeFour = List(3, 4)
val oneTwoThreeFour = oneTwo ::: threeFour
```
 
Метод ::: используется для конкатенации списков.
 
Метод :: вставляет новое значение вначале списка.
Example:
```scala
val twoThree = List(2, 3)
val oneTwoThree = 1 :: twoThree
```
 
Простой способ определения пустого списка – Nil:
`val list = Nil`

Простой способ определения списка – разделить его значения :: и в конце добавить Nil:
`val oneTwoThree = 1 :: 2 :: 3 :: Nil`
 
Пример:
```scala
var jetSet = Set("Boeing", "Airbus")
jetSet += "Lear"
println(jetSet.contains("Cessna"))
```
 
По умолчанию создается immutable set. Во второй строке создается новый set с 3-мя элементами и присваивается переменной, += является сокращением, разворачивающимся в конструкцию:
`jetSet = jetSet + "Lear"`
 
В случае mutable set к нему просто прибавляется новый элемент.
Для создания mutable set нужно его явно импортировать:
```scala
import scala.collection.mutable.Set
val jetSet = Set("Boeing", "Airbus")
jetSet += "Lear"
println(jetSet)
```
 
Для mutable set определен метод += добавляющий новые элементы в set.
 
Пример mutable:
```scala
import scala.collection.mutable.Map
val treasureMap = Map[Int, String]()
treasureMap += (1 -> "Go to island.")
treasureMap += (2 -> "Find big X on ground.")
treasureMap += (3 -> "Dig.")
println(treasureMap(2)
```
 
Метод -> возвращает tuple2 и передает его в метод +=.
 
Пример immutable:
```scala
val romanNumeral = Map(1 -> "I", 2 -> "II", 3 -> "III", 4 -> "IV", 5 -> "V")
println(romanNumeral(4)
```

## Неизменяемые
```scala
import scala.collection.immutable._
List[+A], Vector[+A], Stream[+A], Set[+A], Map[K, +V]
```

Состояние неизменно
Эффективное создание копий при изменении
Hashable, могут храниться в Set, выступать ключами в Map
Ковариантны

### List
Является реализацией Seq по умолчанию
`List[A]` - Связный конечный список. Легко добавить элемент в начало. 
List -  это алгебраический тип с двумя вариантами: 
- `::(head, tail)` - ячейка списка называется “cons”
- `Nil` - пустой список

Определение
`val l = 12 :: 34 :: Nil // или +:`

Операция :: право-ассоциативная то есть начало списка будет с 12 и закончится 34

Два списка разделяют общий хвост
```scala
val lis = 5 :: 4 :: 3 :: 2 :: 1 :: Nil
val lis2 = 10 :: lis
val lis2 = 15 :: lis
```

Декомпозиция на голову и хвост не порождает новых объектов в памяти. List может менять ListBuffer - “конструктор” списка.

#### Проблемы List
- Занимает в два раза больше массива
- Много элементов - нагрузка на сборщик мусора
- Время выполнения многих операций пропорционально длине O(N)
- Вставка в конец - только с полным копированием (список раскручивается в стек и потом стек собирается в новый список)
- средства диагностики heap dump плохо работают

#### Методы

`+:`  - добавление в начало списка
`:+` - добавление в конец списка
```scala
val l = List(1, 2, 3)
l match {
  case head :: tail =>
    println(head)
    println(tail)
  case Nil => // Seq()
    println("Empty!")
}

val res = l match {
  case Seq(one)           => s"only $one"
  case head +: 10 +: tail => (head +: tail).mkString
  case el +: _            => s"first $el and more"
  case Nil                => l
}
```

map применяет функцию к каждому элементу списка
`l.map(List.fill(2)(_))`

flatten превращает список списков в один список
`l.map(List.fill(2)(_)).flatten`

flatMap комбинирует map & flatten
`l.flatMap(List.fill(2)(_))`

collect совмещает map, filter & pattern matching
```scala
val lo = List(Some(1), None, Some(3))
lo collect { case Some(value) => value }

// та же реализация через for
for { Some(i) <- lo} yield i
```

foldLeft - применяет функцию к начальному значению и элементам списка последовательно
`def foldLeft[B](z: B)(op: (B, A) => B): B`
#### Задачки
Посчитать сумму элементов
```scala
def sum(list: List[Int]): Int = list match {
  case Nil => 0
  case head :: tail => head + sum(tail)
}
```
Такая декомпозиция не создает новых элементов.
Сборщик мусора будет собирать список прямо в процессе итерации, если на него нет ссылок.

Получить хвост с N-го элемента
```scala
@tailrec // аннотация проверяет является ли наша рекурсия хвостовой
def drop(l: List[Int], n: Int): List[Int] = {
    if (n > 0 && l.nonEmpty) { drop(l.tail, n - 1) }
    else l
}
```

Реверс листа
```scala
def rev(l: List[Int]): List[Int] = {
    def r(target: List[Int], source: List[Int]): List[Int] = source match {
        case head :: tail => r(head :: target, tail)
        case Nil          => target
    }
    r(Nil, l)
}
```

Получение списка четных элементов
```scala
val rnd = List.fill(10) { Random.nextInt(10) }
val odd = rnd.filter(_ % 2 == 0)
```

Реализация фильтра
```scala
def filter(l: List[Int], f: Int => Boolean): List[Int] = l match {
    case head :: tail if f(head) => head +: filter(tail, f)
    case _ :: tail               => filter(tail, f)
    case Nil                     => Nil
}
```

Реализация фильтра через ListBuffer
```scala
    def filter(l: List[Int], f: Int => Boolean): List[Int] = {
      val lb = ListBuffer[Int]()
      var current = l
      while (current.nonEmpty) {
        if (f(current.head)) lb += current.head
        current = current.tail
      }
      lb.toList
    }
```

Сумма элементов списка
`l.foldLeft(0)(_ + _)`

Удаляем повторяющиеся элементы из списка
```scala
      l.foldLeft(List.empty[Int])((acc, v) =>
        if (acc.headOption.contains(v)) acc else v :: acc
      ).reverse
```

Подсчет элементов удовлетворяющих условию
```scala
def count(list: List[Int], f: Int => Boolean): Int = list.foldLeft(0) {
    (acc, v) => if (f(v)) acc + 1 else acc
}
```

### Stream
`Stream[A]` - Ленивый связный список (хвост вычисляется только когда он нужен), возможно бесконечный. Легко добавить элемент в начало.

Два вида ячеек:
```scala
Stream.Cons[+A](hd: A, tl: => Stream[A])
Stream.Empty
```

Cons ячейка вычисляет "хвост" при обращении, и сохраняет его. Только до следующего звена.

Пример:
```scala
val s: Stream[Int] = 3 #:: 2 #:: 1 #:: Stream.empty

val stream = Stream(1,2,3,2)
val phrase4 = Stream.empty[String]
val phrase5 = phrase4 :+ "=" :+ "love"
```

Пример реализации map:
```scala
def map(s: Stream[Int], f: Int ⇒ Int): Stream[Int] = {
  if (s.isEmpty) {
    s
  } else {
    f(s.head) #:: map(s.tail, f)
  }
}
```

Функции, обходящие весь список "форсируют" его. Например length или fold.
Stream может быть бесконечным: Фибоначчи - каждое последующее число равно сумме двух предыдущих чисел

```scala
import scala.math.BigInt
lazy val fibs: Stream[BigInt] = BigInt(0) #::
                                BigInt(1) #::
                                fibs.zip(fibs.tail).map { n => 
				  n._1 + n._2 
				}
fibs.take(5).toVector
```
### Vector
Современная  персистентная коллекция лишенная недостатков List.
До 32 элементов - это массив. Когда больше 32 элементов - это дерево каждый элемент, которого содержит 32 под элемента.
`Vector[A]` - Индексированный список. Легко получить элемент по индексу, добавить элемент в начало или конец (одинаково эффективно). Неизменяемый аналог ArrayList.

Стоимость операций - effectively constant (максимум 6 уровней, это достаточно):
- получение элемента по индексу
- добавление в конец
- добавление в начало

Vector - не List:
Итератор вместо декомпозиции (на хвост и голову) для обхода всех элементов
Сборка не добавлением, а через VectorBuilder
Используем готовые функции - они уже оптимизированы

Пример:
```scala
val vec = Vector(1, 2, 3, 4)
val more = vec.find(_ > 2)
vec(5)
vec.lift(1)
val (l, r) = vec.splitAt(vec.length / 2) // разделить вектор на 2 части
val res2 = Vector.fill(10) { Try { 1 / Random.nextInt(5) } } // заполнить вектор 10 значениями
res2.count(_.isSuccess) //  узнать количество элементов удовлетворяющих условию
```

Операция +: эффективна только для вектора!

```scala
val initial = Vector[String]("stepic")
val mid = "scala" +: "+" +: initial
val strings = mid:+ "=" +: "love"
strings.mkString("")

val phrase1 = Vector("+")
val phrase2 =phrase1 :+ "Stepic" // лево-ассоциативная операция
val phrase3 ="Scala" +: phrase2

// соединение 2-х коллекций
val phrase = phrase3 ++ phrase5
phrase.mkString(" ")

// собирать коллекции удобно через builder
val builder = Vector.newBuilder[Int]
builder += 1
builder.result()
```

### Set
`Set[A]` - Набор уникальных элементов.

```scala
val set = Set(1,2,3,2)
val map1 = Map("Москва"->12e6, ("Питер",5e6))
val map2= map1 + ("Волгоград"->1e6)
val map2= map1 + ("Волгоград"->1e6)
```

### Map
`Map[K, V]` - Ассоциативный массив "ключ-значение". HashMap - префиксное дерево с 32 ветвями

```scala
Map("one" -> "first", "two" -> "second", "three" -> "third")
m.get("one") // Some("first")
m.contains("q")
val m1 = m + ("five" -> "fifth") // добавление
val m2 = m - "one" // удаление
val m = Map("black" -> "000").withDefaultValue("na") // withDefaultValue помогает избежать NoSuchElementException если m.get("q")
m.toList
```

map/flatMap/filter/fold - аналогично `Seq[(K,V)]`.
`m.map(p => p._1.toUpperCase -> p._2)`

Ключ - неизменяемый объект любого типа
Если результат не пара, то получим Seq

Метод hashCode возвращает Int для любого объекта
- У равных (equals) объектов они одинаковые
- У неравных - различные, насколько это возможно
- У case class и пар создается автоматически

Добавление и удаление - effectively constant, как у Vector.
Поиск - effectively constant, если хеш-функция хорошая.
Значения с одинаковым хеш-кодом хранятся в списке.

Map - частично определенная функция
Пример:
```scala
val m = Map("one" -> 1, "two" -> 2)
List("one", "two", "three").collect(m)
или
List("one", "two", "three").flatMap(m.get)
```

### View
Делает ленивые вычисления
View - не коллекция
```scala
case class Person(id: Int, name: String, student: Boolean)
    def makeIndex(persons: Vector[Person]): Map[Int, Person] = {
      // каждая операция - новая структура
      persons.filter(_.student).map(p => p.id -> p).toMap
    }
```

решение:
`persons.view.filter(_.student).map(p => p.id -> p).toMap // аналог stream в java`

SeqView - последовательный доступ на базе Iterator
- map/filter/.. выполняются при обращении
- take не вычисляет то что не нужно
- drop вычисляет до начала потом по пеобходимости
- append/prepend/concat эффективно

IndexedSeqView - доступ по индексу
- доступ по индексу
- slice/splitAt/take/drop - делают подколлекции без копирования
- head/tail и декомпозиция

MapView для Map

## Изменяемые
```scala
import scala.collection.mutable._
Buffer[A], Set[A], Map[A, B], Builder[-E,+C]
```

Состояние может меняться.
Эффективны для большого количества операций.
Копирование неэффективно т.к. выделяет новую память.

Пример:
`Buffer[A]` - Саморастущий массив
`ArrayBuffer[A]` - аналог ArrayList из Java.
`Set[A]` - Набор уникальных элементов
`Map[K, V]` - Ассоциативный массив "ключ-значение"
`Builder[E, Coll]` - Промежуточный накопитель для построения коллекции

Пример:
```scala
import scala.collection.mutable._
val strings = Buffer[String]()
strings +="Scala"
strings +="+"
strings +="Stepic"
strings.mkString("")

val buf = ArrayBuffer[Int](1,2,3)
buf += 4 // добавление элемента
buf(1) // получение элемента, может дать IndexOutOfBoundsException
buf.lift(1) // получение элемента Option
```

## Общий надтип
```scala
import scala.collection._
Seq[+A], Set[A], Map[K, +V], Iterator[+A]
```
Для соединения элементов Seq используется оператор +: а для списков ::.
Seq - общий тип для имеющих определенный порядок (списки, массивы, вектора и т.п.)

Проверка длины списка может быть дорогой для структур, которые не знают своей длины, поэтому для этого следует использовать метод nonEmpty:
```scala
val l = List(1,2,3)
l.nonEmpty
```

## Массивы
`Array[A]`
Очень эффективный, но низкоуровневый т.к. имеет специальные версии для примитивов (Int, Long, Double, Boolean). Фиксированного размера.
 
Пример:
```scala
val ints = Array(1,2,3,5)
ints(2)//3
ints(2)=6
ints(2)//6
ints(4)//error

    val arr = Array("h", "e", "l", "l", "o", ".")
    arr(5) = "!"
    println(arr.mkString("-"))

    Array.ofDim[Boolean](2).foreach(println)
```


## Строки
String
Неизменяемые массивы символов. Любое изменение выделяет новую строку.

Пример:
```scala
val lang = "Scala"
val plat = "Stepic"
val course = lang + "" + plat
val course1 = s"$lang $plat" // интерполяция решает проблему создания новых строк
val char: Char = course(3)// 'l'
```

Пример:
Сортировка слиянием хорошо подходит для односвязных списков
```scala
import scala.util.Random
val list = List(2,5,7,1,4)
val randomList = List.fill(Random.nextInt(100))(Random.nextInt(1000))

def merge(as: List[Int], bs: List[Int], acc: List[Int]=Nil): List[Int] = as match {
 case List() => acc.reverse ++ bs
 case a +: reastA => as match {
  case List() => acc.reverse ++ as
  case b +: reastB => 
   if(a<b) merge(reastA, bs, a :: acc)
   else merge(as, reastB, b :: acc)

def mergeSort(as: List[Int]): List[Int] = as match {
 case Nil | (_ :: Nil) => as
 case _ => 
  val (left, right) = as.splitAt(as.length/2)
  val leftSorted = mergeSort(left)
  val rightSorted = mergeSort(right)
  merge(leftSorted, rightSorted)
}

mergeSort(randomList) = randomList.sorted
```

Задание: Получив некоторый произвольный список нулей и единиц, разделите их на два списка.

Результат выведите через запятую, каждый в отдельную строку, первыми выводятся нули.

Ответ:
```scala
val ints: List[Int] = Lesson.ints  
    val tuple = ints.partition(_ == 0)
    println(tuple._1.mkString(","))
    println(tuple._2.mkString(","))
// или
ints.partition(_ == 0).productIterator.foreach(x => println(x.asInstanceOf[List[Int]].mkString(",")))
// или
val sorted = ints.sorted
    val (zeroList, oneList) = sorted.splitAt(sorted.indexOf(1))
    println(zeroList.mkString(","))
    println(oneList.mkString(","))    
// или
println(ints.sorted.takeWhile(_==0).mkString(","))
    println(ints.sorted.dropWhile(_==0).mkString(","))
// или
val sorted = ints.partition(_ == 0) match {
        case (x, y) => s"${x.mkString(",")}\n${y.mkString(",")}"
        case x => s"$x"
    }
```

Задание: В данной задаче необходимо реализовать алгоритм нахождения k-ой порядковой статистики, мат.ожидание времени работы которого составляет O(n). Для этого реализуйте метод kOrder (его сигнатура в шаблоне).

На вход в первой строке подаётся k ﻿- номер порядковой статистики, которую надо найти. Во второй строке - элементы набора.

Ответ:
```scala
val k: Int = StdIn.readLine().toInt
  val list: List[Int] = StdIn.readLine().split(" ").map(_.toInt).toList  
  println(kOrder(list, k-1))
    
  @scala.annotation.tailrec    
  def kOrder(list: List[Int], k: Int): Int = {

    val pivot = list(list.length / 2)

    val tmp  =  list.partition( _ <= pivot )
    val left  = tmp._1
    val right = tmp._2

    if(left.length == k+1 ) {
      pivot
    } else if( left.length < k+1 ){
      kOrder(right, k - left.length )
    } else {
      val tmp2 = left.partition( _ < pivot )
      val left2  = tmp2._1
      if(left2.length < k+1 ) {
        pivot
      }else {
        kOrder(left2, k )
      }
    }
  }
}
// или
def kOrder(list: List[Int], k: Int): Int = {
      val head +: rest = list
      val (sm, hi) = rest.partition(_ <= head)
      sm.length match {
        case x if x == k - 1 => head
        case x if x > k - 1 => kOrder(sm, k)
        case _ => kOrder(hi, k - sm.length - 1)
      }
    }
      val k = readInt()
      print(kOrder(readLine().split(" ").toList.map(_.toInt),k))
}
// или
val number = StdIn.readInt()
    val list: List[Int] = StdIn.readLine().split(" ").map(_.toInt).toList

// val list = Option(scala.io.StdIn.readLine()).toList.flatMap(_.split(" ")).map(_.toInt)

    println(kOrder(list, number))

    def kOrder(list: List[Int], k: Int): Int = {
      val sorted = list.sorted
      sorted(k - 1)
    }
// def kOrder(list: List[Int], k: Int): Int = list.sortWith(_<_)(k-1)
}
```

## Отображения
```scala
val nums = List.range(0, 10)  
val alfa = 'A' to 'Z'  
val nums2 = nums.map(i=>alfa(i))  
val nums3 = nums.map(alfa) // List[Char]
```
## Операции для последовательностей Vector, Stream, List:

### Получение элемента
```scala
val cities = Vector("Москва", "Волгоград", "Питер")
cities(1)//Волгоград
cities.head //Москва
cities.last //Питер

val cityMap = Map("Москва"->12e6, "Питер"->5e6)
cityMap("Питер")//5000000.0
cityMap.get("Москва")//Some(1.2E7)
cityMap.get("Петушки")//None

val citySet = Set("Москва", "Волгоград", "Питер")
citySet("Волгоград")//true когда элемент входит в Set
citySet("Петушки")//false
```

### Информация
```scala
val cities = Vector("Москва", "Волгоград", "Питер")
cities.size //3
cities.contains("Москва") //true есть ли элемент в последовательности
cities.indicies //0 until 3 диапазон индексов

val cityMap = Map("Москва"->12e6, "Питер"->5e6)
cityMap.size //2
cityMap.contains("Питер") //true есть ли такой ключ
cityMap.keySet // Set("Москва", "Питер") набор всех ключей

val citySet = Set("Москва", "Волгоград", "Питер")
citySet.size //3
citySet.contains("Петушки") //false
```

## Субколлекции
```scala
val nums = Vector.range(1,11)
nums.slice(3,7) // получает все элементы с индекса 3 по 6
nums.tail // все элементы кроме первого
nums.init // все элементы кроме последнего
nums.take(3) // первые 3 элемента
nums.drop(3) // все элементы кроме первых 3х
nums.takeRight(3) // последние 3 элемента
nums.dropRight(3) // все элементы кроме последних 3х
```
Для неупорядоченных коллекций
```scala
val cityMap = Map("Москва"->12e6, "Питер"->5e6, "Волгоград"->1e6)
cityMap - "Питер" // удалить 1 элемент
cityMap -- List("Москва", "Волгоград") // удалить несколько элементов

val citySet = Set("Москва", "Волгоград", "Питер")
citySet - "Питер"
citySet -- List("Москва", "Волгоград")
```

## Условный отбор
```scala
val nums = Vector.range(1, 21)
val odds = nums.filter(_ % 2 == 1)
val evens = nums.filterNot(_ % 2 == 1)
val (odds1, evens1) = nums.partition(_ % 2 == 1) // вернет кортеж из filter и filterNot
val small = nums.takeWhile(_<10) // вернет префикс удовлетворяющий условию
val big = nums.dropWhile(_<10) // вернет все элементы, кроме префикса удовлетворяющего условию
val (small1, big1) = nums.span(_<10) // вернет кортеж из takeWhile и dropWhile
```

## Map flatMap filter
Смысл map заключается в том, что заданная функция применяется к каждому элементу списка.

flatMap очень похож на map, только он преобразует каждый элемент в целый список элементов и выполняет действия уже с ними, а потом результат собирает в одно целое.

Пример:
```scala
  val fruits = List("apple", "banana")
  
  val mapResult = fruits.map(_.toUpperCase)
  val flatResult = fruits.flatMap(_.toUpperCase)
  
  println(mapResult) // List(APPLE, BANANA)
  println(flatResult) // List(A, P, P, L, E, B, A, N, A, N, A)
```

Именно из-за того, как работает flatMap, если нам требуется проставить точку после каждого символа строки и на выходе получить модифицированную строку, использовать придется именно его.
```scala
  val s = "Hello"
  val newStr: String = s.flatMap(c => (c + "."))
  println(newStr) // H.e.l.l.o.
```

map тоже сработает, только вернет уже не строку
`  println(s.map(c => (c + "."))) // ArraySeq(H., e., l., l., o.)`
 
Комбинатор flatMap соединяет в себе map и flatten.
Комбинируя map и flatMap, мы получаем возможность пройтись по списку. Альтернативой является применение for:
```scala
    val list1 = List(1, 2)
    val list2 = List("a", "b")
    val comb = list1.flatMap(l => list2.map(_ + l))

    val f = for {
      l1 <- list1
      l2 <- list2
    } yield l2 + l1
```

Пример filter:
```scala
    val list1 = List(1, 2)
    val list2 = List("a", "b")
    val comb = list1.filter(_ > 1).flatMap(l => list2.map(_ + l))

    val f = for {
      l1 <- list1 if l1 > 1
      l2 <- list2
    } yield l2 + l1
```

## Collect
Метод collect делает и filter и map с помощью частичной функции в качестве которой используем выражения паттерн матчинга:
```scala
 nums.collect {
  case i if i%2==0 => alfa(i/2*3)
  case 3 => '_'
  case 5 | 7 => '!'
 }
```

## flatten
Производит конкатенацию вложенных списков
```scala
val charLists: List[List[Char]] = 
 nums.map(i => List(alfa(i), alfa(i+3)))
val charLists: List[Char]=charLists.flatten 
```

### Примеры
Является ли число простым:
```scala
def isPrime(x: Long): Boolean =
  Stream.from(2).takeWhile(p=>p*p<x).forall(x%_ != 0)
isPrime(7)

lazy val primes: Stream[Long] = 2L #:: Stream.iterate(3L)(_+2L).filter(isPrime)
// 3L - с чего начали
// _+2L - шаг
primes.take(50).force

def isPrime(x: Long): Boolean =
  primes.takeWhile(p=>p*p<x).forall(x%_ != 0)
```

В новых версиях Scala коллекцию Stream переименуют в lazyList

Задание: В переменной list лежит отсортированный в порядке возрастания список целых чисел. Со списком необходимо выполнить следующие операции:

Взять все числа меньше 100 (список может быть большим)
Выбрать все числа, которые делятся на 4
Поделить их на 4
Вывести на экран в отдельной строке каждый элемент, кроме последнего

Ответ:
```scala
val list = List(10, 20, 30, 40, 50, 60, 70, 80, 90, 100, 110, 120, 130, 140, 150)
list.filter(_<100).filter(_%4==0).map(_/4).init.map(println)
// или
list
    .collect{case x if x < 100 & x % 4 ==0 => x / 4}
    .dropRight(1)
    .foreach(println)
// или
list.filter(tmp => tmp < 100 && tmp % 4 == 0)
    .map(tmp => tmp / 4)
    .init
    .foreach(println)
```

Задание: Считайте числа из консоли до слова END.
С полученным списком необходимо выполнить:
выбрать каждый второй элемент
каждый выбранный элемент умножить на 2
вывести в консоль сумму элементов полученного списка
Рекомендация: для считывания в список можно использовать Stream

Ответ:
```scala
println(Stream.iterate(readLine())(_ => readLine())
      .takeWhile(_ != "END")
      .zipWithIndex
      .collect { case (x, i) if (i + 1) % 2 == 0 => x.toInt * 2 }.sum)
// или
val x = Stream.continually(StdIn.readLine)
      .takeWhile(_ != "END")
      .map(_.toInt)
      .zipWithIndex
      .collect { case (v, i) if i % 2 == 1 => v }
      .sum * 2

    println(x)
// или
println(Stream.continually(readLine()).takeWhile(_ != "END").zipWithIndex.collect { case (elem, ind) if ind % 2 == 1 => elem.toInt * 2 }.sum)
// или
lazy val strings = Stream.continually(readLine)
        .takeWhile(_ != "END")

      val sum = List.range(1, strings.length, 2)
        .map(strings)
        .map(_.toInt * 2)
        .sum

      println(sum)
// или
val lines: Stream[String] = Stream.continually(StdIn.readLine())
    val filtered: Stream[String] = lines.takeWhile(_ != "END")

    val eachSecond: Stream[String] = filtered.zipWithIndex.filter(_._2 % 2 != 0).map(_._1)

    val sum = eachSecond.map(_.toInt).sum

    println(sum * 2)
// или
val sum = Stream.continually(StdIn.readLine())
      .takeWhile(_ != "END")
      .grouped(2)
      .filter(_.size == 2)
      .map(_.last)
      .map(_.toInt)
      .map(_ * 2)
      .sum
    println(sum)
// или
println(Stream.continually(readLine())
        .takeWhile(_ != "END")
        .zipWithIndex
        .filter(_._2%2 != 0)
        .map(_._1.toInt).sum*2)
// или
val sumOfReads = Iterator
      .continually(scala.io.StdIn.readLine)
      .takeWhile(_ != "END")
      .zipWithIndex
      .withFilter(_._2 % 2 == 1)
      .map(_._1.toInt * 2)
      .reduceLeftOption(_ + _)
      .getOrElse(0)
    println(sumOfReads)
// или
Stream
        .continually(StdIn.readLine())
        .takeWhile(_ != "END")
        .map(_.toInt)
        .zipWithIndex
        .filter(_._2 % 2 == 1)
        .unzip
        ._1
        .map(_ * 2)
        .sum
// или
def stdins: Stream[String] = readLine() match {
      case s if s == null => Stream.empty
      case s => s #:: stdins
      }

      val input: List[String] = stdins.takeWhile(x => x != "END").force.toList
      println(input.filter(x => input.indexOf(x) % 2 == 1).map(x => x.toInt * 2).sum) 
// или
val input = Stream.continually(StdIn.readLine).takeWhile(_ != "END")
    println(input.map(_.toInt).drop(1).grouped(2).map(_.head * 2).foldLeft(0)(_+_))
// или
io.Source.stdin.getLines
          .takeWhile(_ != "END")
          .zipWithIndex
          .filter(_._2 % 2 == 1)
          .map(_._1.toInt * 2)
          .sum)
// или
Stream.iterate((0, readLine()))(x => (x._1 + 1, readLine()))
      .takeWhile(_._2 != "END")
      .filter(_._1 % 2 != 0)
      .map(_._2.toInt * 2)
      .sum
// или
var line = StdIn.readLine()
    val ints = Buffer[Int]()
    while (line != "END") {
      ints += line.toInt
      line = StdIn.readLine()
    }
    println(List.range(0, ints.size).filter(_ % 2 == 1).map(ints(_) * 2).sum)
// или
(stdin.getLines.takeWhile( _ != "END" ) zip Stream.from(1).toIterator).filter(_._2 % 2 == 0).map(_._1.toInt * 2).sum
// или
val list = io.Source.stdin.mkString.split("\n").toList            
    val result = list.takeWhile(_ != "END").map(_.toInt).zipWithIndex.filter(_._2 % 2 == 1).map(_._1 * 2).sum
// или
val input = Source.fromInputStream(System.in)   
    val res = input.getLines().takeWhile( str => str != "END")
      .map(_ toInt )
      .zipWithIndex
      .filter(_._2 % 2 != 0)
      .map(_._1*2).sum
    println(res) 
```

Задание: Некоторые генетические алгоритмы для генерации новых хромосом из старых используют приём под названием кроссинговер.

Будем представлять хромосому с генами [xxxxx]   в виде списка List('x', 'x', 'x', 'x', 'x') . Тогда суть приёма заключается в следующем:

Берутся две хромосомы одинаковой длины, например [xxxxx] и [yyyyy]. Списки для них будут выглядеть так:
List('x', 'x', 'x', 'x', 'x')
List('y', 'y', 'y', 'y', 'y')

Выбираются так называемые `точки кроссинговера`. В нашем случае это некоторые индексы в диапазоне [1, длина списка генов хромосомы]. Пусть выбраны индексы 1 и 3.
Для  каждого индекса, по возрастанию, хромосомы обмениваются своими частями, стоящими после этого индекса. В  нашем случае после кроссинговера по индексу 1:
List('x', 'y', 'y', 'y', 'y')
List('y', 'x', 'x', 'x', 'x')
А после дальнейшего кроссинговера по индексу 3:
List('x', 'y', 'y', 'x', 'x')
List('y', 'x', 'x', 'y', 'y')
Ничего считывать из ﻿консоли не надо. ﻿Вам даны:
val points: List[Int] = Lesson.points // точки кроссинговера
val chr1: List[Char] = Lesson.chr1 // первая хромосома
val chr2: List[Char] = Lesson.chr2 // вторая хромосома
Выведите результат хромосомы после кроссинговера, сначала первую, затем вторую. Без пробелов и знаков препинания.

Ответ:
```scala
def iter(last: Int, p: List[Int], f: Char, s: Char, res1: List[Char], res2: List[Char]): (List[Char], List[Char]) = p match {
  case Nil => (res1.take(last) ++ Array.fill(res1.length - last)(s).toList, res2.take(last) ++ Array.fill(res2.length - last)(f).toList)
  case h :: tail if last == 0 =>
    iter(h, tail, f, s, res1, res2)
  case h :: tail =>
    iter(h, tail, s, f, res1.take(last) ++ Array.fill(res1.length - last)(s).toList, res2.take(last) ++ Array.fill(res2.length - last)(f).toList)
}
val (l, r) = iter(0, points, chr1.head, chr2.head, chr1, chr2)
println(l.mkString(""))
println(r.mkString(""))
или
def crossover[T](points: List[Int], x: List[T], y: List[T]): (List[T], List[T]) =
      points match {
        case Nil => (x, y)
        case head :: tail =>
          val (a1, a2) = x.splitAt(head)
          val (b1, b2) = y.splitAt(head)
          val (newX, newY) = (a1 ::: b2, b1 ::: a2)
          crossover(tail, newX, newY)
      }


    val (crs1, crs2) = crossover(points, chr1, chr2)

    println(crs1.mkString(""))
    println(crs2.mkString(""))
или
def crossingOver(first: List[Char], second: List[Char], start: Int) =
      first.zip(second).zipWithIndex.map( x => if (x._2 >= start) x._1.swap else x._1).unzip

    points.foreach(i => {  sequences = crossingOver(sequences._1, sequences._2, i) })

    println(sequences._1.mkString(""))
    println(sequences._2.mkString(""))
или

      
    def out(chr: List[Char]) = {
        println(chr.mkString(""))
    }
      
    val res = points.foldLeft((chr1, chr2))(cross)
    out(res._1)
    out(res._2)
или
def change(a: List[Char], b: List[Char], indexs: List[Int]): (List[Char], List[Char]) = {
      indexs match {
        case Nil => (a, b)
        case _ => change(a.take(indexs.head) ++ b.drop(indexs.head), b.take(indexs.head) ++ a.drop(indexs.head), indexs.drop(1))
      }
    }

    val result = change(chr1, chr2, points)
или
val (firstOut, secondOut) = (0 :: points).zip(points :+ chr1.size)
      .map { case (start, end) => chr1.slice(start, end).zip(chr2.slice(start, end)) }
      .zipWithIndex
      .flatMap { case (data, i) => if (i % 2 == 0) data else data.map(_.swap)}
      .unzip
или
def cross(l1: List[Char], l2: List[Char], p: Int): (List[Char], List[Char]) = 
      (l1.take(p) ::: l2.drop(p)) -> (l2.take(p) ::: l1.drop(p))
      
    val (r1, r2) = points.foldLeft(chr1 -> chr2)((ll, p) => cross(ll._1, ll._2, p))
или
def crossOver(list: List[(Char, Char)], pts: List[Int]): List[(Char, Char)] =
      pts match {
        case List() => list
        case head :: tail =>
          crossOver(list.take(head) ++ list.drop(head).map(_.swap), tail)
      }


    val (chr1After, chr2After) = crossOver(chr1.zip(chr2), points).unzip
```

Задание:
 List - одна из любимых коллекций скалистов. Её иммутабельность играет на руку при написании параллельных программ, а её API позволяет эффективно работать с элементами, лежащими в начале коллекции. С задачей добавления одиночных элементов в начало она справится хорошо (за константное время), так как реализована на основе односвязного списка, но вот ассимптотическая оценка операции добавления таких элементов в конец вырастет до O(n) , где n - длина списка (подробнее, почему List реализован на односвявзных списках, и почему так происходит можно почитать тут).

 И тут на сцену выходит структура данных под названием Difference List.

 Рассмотрим две операции над списками: операцию .prepend добавления элементов в начало списка и операцию .append добавления элементов в конец списка. Каждую такую операцию можно рассматривать как некоторую функцию List[A] => List[A], где A - тип элементов списка, тогда некоторая цепочка таких операций - это композиция таких функций. Именно за счёт такого приёма DiffList-ы позволяют избавиться от дорогостоящего добавления в конец, заменяя его на добавление в начало. Поговорим подробнее, как это происходит.

 Ваша реализация DiffListImp[A] должна реализовывать следующий интерфейс:

```scala
abstract class DiffList[A](calculate: List[A] => List[A]) {
  def prepend(s: List[A]): DiffList[A]

  def append(s: List[A]): DiffList[A]

  def result: List[A]
}
```
  Вам необходимо реализовать prepend и append и result:

Метод prepend принимает на вход список s, добавляемый в начало. Вернуть надо новый объект, сконструировав его от новой функции. Эта функция должна возвращать список s , добавленный в начало к списку, возвращаемому из calculate.
Метод append принимает на вход список s, добавляемый в конец. Вернуть надо новый объект, сконструировав его от новой функции. Эта функция должна возвращать результат применения функции calculate к конкатенации списка  s и аргумента этой функции.
Метод result применяет все накопленные операции и отдаёт итоговый список.
 Как выглядит использование DiffListImpl:
```scala
val l1 = List(1,2,3)
val l2 = List(4,5,6)
val dl = new DiffListImpl[Int](identity)

val result = dl.append(l2).prepend(l1).result // List(1,2,3,4,5,6)
```
 Ваша задача - реализовать недостающие методы интерфейса, ничего считывать из консоли и писать в неё не надо.

Справка: сконструировать новый объект можно так: ﻿

```scala
val f = identity[List[Int]]
val obj = new DiffListImpl(f)
```

Ответ:
```scala
final class DiffListImpl[A](listFunc: List[A] => List[A]) extends DiffList[A](listFunc) {
  var l: List[A] = Nil
  def prepend(s: List[A]) = {
    l = listFunc(s ++ l)
    this
  }
  def append(s: List[A]) = {
    l = listFunc(l ++ s)
    this
  }
  def result = l
}
или
final class DiffListImpl[A](listFunc: List[A] => List[A]) extends DiffList[A](listFunc) {
  def prepend(s: List[A]) = {
    new DiffListImpl[A]( x =>  listFunc( s ::: x)  )
  }

  def append(s: List[A]) = {
    new DiffListImpl[A]( x =>  listFunc(x ::: s)  )
  }

  def result = {
    listFunc(Nil)
  }
}
```

# Алгебраические типы данных (ADT)

Типы ADT:
- Sum ADT (тип суммы)
- Product ADT (тип произведения)
- Hybrid ADT (гибридный тип)

## Sum ADT
Sum ADT применяется тогда, когда мы можем просто перечислить все возможные варианты типа.

Сумма типов A + B - это такой тип, который позволит закодировать все значения типа A и все значения типа B

Either - это наиболее общий способ кодировать суммы в Scala

Пример:
```scala
sealed trait Either[+A, +B]

final case class Right[+A, +B](value: B) extends Either[A, B]
final case class Left[+A, +B](value: A) extends Either[A, B]
```

Пример:
```scala
type BooleanOrUnit = Either[Boolean, Unit]

val v1: BooleanOrUnit = Left(true)
val v2: BooleanOrUnit = Left(false)
val v3: BooleanOrUnit = Right(())
```

Пример реализовать тип PaymentMethod:
```scala
type CreditCard
type WireTransfer
type Cash

type PaymentMethod = Either[CreditCard, Either[WireTransfer, Cash]]

Left[Creditcard]
Right[Left[Wire]]
Right[Right[Cash]]

// or
sealed trait PaymentMethod
case object CreditCard extends PaymentMethod
case object WireTransfer extends PaymentMethod
case object Cash extends PaymentMethod
```

Например, необходимо организовать работу с днями недели. Мы точно знаем, что всего существует семь дней недели, которые вполне возможно перечислить:
```scala
sealed trait WeekDay
case object Mon extends WeekDay
case object Tue extends WeekDay
case object Wed extends WeekDay
case object Thu extends WeekDay
case object Fri extends WeekDay
case object Sun extends WeekDay
case object Sat extends WeekDay

val day: WeekDay = Mon
println(s"Today is $day") // Today is Mon
```

## Product ADT
В случае Product ADT становится невозможно перечислить все возможные значения т.к. их слишком много. Раз просто перечислить нельзя - значит, нельзя для каждого возможного значения предоставить отдельный конструктор и придется организовывать код так, чтобы ограничится одним конструктором.

Произведение типов A * B - это такой тип, который позволит закодировать все возможные комбинации типов A и B

Tuple(кортеж) - наиболее общий способ кодировать типы произведения (максимальное количество элементов - 22), очень похож на case class, но нет именованных полей:
- Конструктор / деконструктор
- Структурное сравнение
- Хэш код
- Копирование
- Строковое представление

Пример создать возможные экземпляры с типом ProductUnitBoolean:
```scala
type ProductUnitBoolean = (Unit, Boolean)

lazy val p1: ProductUnitBoolean = ((), true)
lazy val p2: ProductUnitBoolean = ((), false)
```

Пример реализовать тип `CreditCard` который может содержать номер (String), дату окончания (java.time.YearMonth), имя (String), код безопастности (Short):
```scala
type CreditCard = (String, YearMonth, Short)
```

Если бы перед нами была поставлена задача закодировать все возможные значения RGB цветов посредством Sum ADT, нам бы потребовалось 256^3 различных конструкторов. 

```scala
  sealed case class RGB(red: Int, green: Int, blue: Int)

  val whiteRGB = RGB(255, 255, 255)
  println(s"White: $whiteRGB") // White: RGB(255,255,255)
```

## Hybrid ADT
Наиболее часто работа все же происходит с Hybrid ADT.
UnexpectedValueForFieldCoproductHint(Unquoted("PoS"))
UnexpectedValueForFieldCoproductHint(Quoted("PoS"))

Как можно догадаться, Hybrid ADT объединяет Sum ADT и Product ADT:
```scala
  sealed trait Platform
  case class IOS(appId: String) extends Platform
  case class Android(packageId: String, sha1Cert: String) extends Platform
```
В данном случае мы имеем дело с гибридным типом, т.к. работа идет с платформой iOS или Android ("или" указывает на Sum ADT);
Product ADT был задействован при описании того же Android т.к. пришлось создавать универсальный конструктор, учитывающий различные значения для packageId и sha1Cert.

Еще примеры

Фиксированная схема данных и произвольные операции, тогда Pattern Matching:
Product - тип можно разобрать на пары, а потом обратно собрать.
```scala
sealed trait Expr extends Product with Serializable
final case class Number(value: Int) extends Expr
final case class Plus(l: Expr, r: Expr) extends Expr
final case class Minus(l: Expr, r: Expr) extends Expr
def value(expr: Expr): Int = expr match {
  case Number(v) => v
  case Plus(l, r) => value(l) + value(r)
  case Minus(l, r) => value(l) - value(r)
}
val res = value(Plus(Number(1), Number(4)))
```

Конструкция Product with Serializable необходимо чтобы была возможность добавить такой элемент в Buffer.

Пример:
```scala
val number = Number(3)
val expr = Plus(Number(2), Number(3))
val buf = ArrayBuffer(Number(2), expr)
buf += number
```

Конструкция final case class запрещает наследование помогает компилятору не искать что-то где-то еще.

или полиморфизм, когда фиксированные операции и большое разнообразие объектов:
```scala
trait Expr {
  def eval: Int
}
case class Number(value: Int) extends Expr {
  override def eval = value
}
case class Plus(l: Expr, r: Expr) extends Expr {
  override def eval = l.eval + r.eval
}
case class Minus(l: Expr, r: Expr) extends Expr {
  override def eval = l.eval - r.eval
}
Plus(Number(1),Number(4)).eval
```

# Линеаризация
Процесс инициализации противоположен процессу линеаризации

Пример:
```scala
class A {
    def foo() = "A"
  }

  trait B extends A {
    override def foo() = "B" + super.foo()
  }

  trait C extends B {
    override def foo() = "C" + super.foo()
  }

  trait D extends A {
    override def foo() = "D" + super.foo()
  }

  trait E extends C {
    override def foo(): String = "E" + super.foo()
  }

  // A -> D -> B -> C
  val v = new A with D with C with B

  // A -> B -> C -> E -> D
  val v1 = new A with E with D with C with B

```

# Value classes
Value classes - на уровне кода создают типизированные обертки повышают типобезопасность. Класс и объект не создается в рантайме.
- Позволяют избежать дополнительной аллокации объектов во время исполнения
- Один параметр конструктора
- Тело может содержать только методы
- Миксировать можно только universal trait
- Наследуются от AnyVal

class Id(val raw: String): extends AnyVal

Когда случаются аллокации
- Value class используется как другой класс
- Сохраняется в массив
- Применяется проверка типа в массиве (isInstanceOf или паттерн матчинг)

# Universal traits
Universal traits - если для Value класса хотим подмешать трейт то можно подмешать только Universal
- могут содержать только def методы
- не имеют инициализации
- наследуются от Any

trait Printable extends Any

# Чистые функции
Чистые функции (pure function):
1. Являются детерминированными (при подаче на вход аргумента всегда дают определенный результат, который не меняется со временем) 
2. Не обладают побочными эффектами (запись в файл, вывод на экран, взаимодействие с сетью и т.п.)

Вычисление чистых функций:
1. Можно кешировать
2. Переупорядочивать и откладывать их выполнение
3. Передавать в другие потоки

Pure functions - функции, которые для одних и тех же аргументов вычисляют одни и те же результаты и не делают никаких сайд эффектов, то есть функция не делает никаких сайд эффектов кроме как вычисление результата в зависимости от переданного на вход значения

f: A => B - функция, которой на вход подают тип А, а на выходе получаем тип В - выполняет вычисление которое связывает каждое значение а типа А только с одним значением b типа B. 
Пример: 
intToString: Int => String
def sum(x: Int, y: Int): Int = x + y - чистая функция
Функция возвращающая длину строки (строки неизменяемы в Java и Scala)

Чистые функции позволяют делать кеширование, код становится более понятным, их легко тестировать, переиспользовать, выполнять параллельно, обобщать и они менее подвержены ошибкам

Функция чистая, если вызывая ее с ссылочно-прозрачными аргументами она также является ссылочно-прозрачной
> Функция f чистая, если выражение f(x) ссылочно-прозрачное для всех ссылочно-прозрачных х

Сайд эффекты:
- изменение значения переменной
- изменение структуры данных
- изменение значения в объекте
- выбрасывание исключения или остановка с ошибкой
- вывод в консоль или чтение пользовательского ввода
- чтение/вывод в файл
- рисование на экране

Нечистая функция или процедура - функция выполняющая сайд-эффекты

Пример нечистой функции
```scala
class Cafe {
	def buyCoffee(cc: CreditCard): Coffee = {
		val cup = new Coffee()
		cc.charge(cup.price) // сайд эффект
		cup
	}
}
```

cc.charge - имеет сайд эффект так как оплата кредитной картой подразумевает взаимодействие с внешним миром. Такой код сложно тестировать.

перепишем
```scala
class Cafe {
	def buyCoffee(cc: CreditCard, p: Payments): Coffee = {
		val cup = new Coffee()
		p.charge(cc, cup.price) // сайд эффект
		cup
	}
}
```

У нас все еще есть сайд эффект, но мы восстановили тестируемость так как теперь можно замокать интерфейс Payments

версия с чистой функцией
```scala
class Cafe {
	def buyCoffee(cc: CreditCard): (Coffee, Charge) = {
		val cup = new Coffee()
		(cup, Charge(cc, cup.price))
	}
}
```

теперь мы возвращаем кофе и объект Charge
```scala
case class Charge(cc: CreditCard, amount: Double) {
	def combine(other: Charge): Charge =
		if (cc == other.cc)
			Charge(cc, amount + other.amount)
		else
			throw new Exception("Can't combine charges to different cards")
}
```

напишем функцию для заказа нескольких чашек кофе, используя buyCoffee
```scala
def buyCoffees(cc: CreditCard, n: Int): (List[Coffee], Charge) = {
	val purchases: List[(Coffee, Charge)] = List.fill(n)(buyCoffee(cc))
	val (coffees, charges) = purchases.unzip
	(coffees, charges.reduce((c1,c2) => c1.combine(c2)))
}
```

# Referential transparency (RT)
Ссылочная прозрачность - когда выражение может быть заменено его результатом не изменяя результата программы
> выражение е ссылочно-прозрачное, если для всех программ р, можно заменить е его результатом (телом функции) без изменения результатов р.

В любой программе, выражение считается referentially transparent, если мы можем
заменить это выражение на его результат, при этом значение программы не изменится.

Пример:
Чистые функции

Пример:
```scala
val x = "Hello, World"
val r1 = x.reverse // dlroW ,olleH
val r2 = x.reverse // dlroW ,olleH

val r1 = "Hello, World".reverse // dlroW ,olleH
val r2 = "Hello, World".reverse // dlroW ,olleH
// строка х ссылочно-прозрачная

Пример:
val x = new StringBuilder("Hello")
val y = x.append(", World") // Hello world
val r1 = y.toString() // Hello, world
val r2 = y.toString() // Hello, world

val r1 = x.append(", World").toString() // Hello, world
val r2 = x.append(", World").toString() // Hello, world, world

// StringBuilder.append не чистая функция
```

# Рекурсия
Рекурсивная функция - это функция, частью тела которой является вызов самой себя.

2 правила:
- условие выхода
- рекурсивный шаг

Пример факториал:
```scala
  def factRec(n: Int): Int =
    if(n <= 0) 1 else n * factRec(n - 1)

  def factRecTail(n: Int): Int = {
    @annotation.tailrec
    def loop(i: Int, acc: Int): Int = {
      if(i <= 0) acc
      else loop(i - 1, i * acc)
    }
    loop(n, 1)
  }
```

Пример Фибоначчи:
F0 = 0, F1 = 1, Fn = Fn-1 + Fn - 2
0, 1, 1, 2, 3, 5
```scala
   def fibRec(n: Int): Int = {
    if(n==0) 0
    else if(n==1) 1
    else fib(n-1)+fib(n-2)
   }

   def fibRecTail(n: Int): Int = {
    def go(n: Int, acc: Int, x: Int): Int = n match {
      case 0 => acc
      case _ => go(n-1, x, acc+x)
    }
    go(n, 0, 1)
   }

def fib (n:Int) : Int = {
	@annotation.tailrec
	def go(n:Int, prev:Int, acc:Int) : Int = {
		if (n <= 1) acc
		else go(n-1, acc, prev+acc)
	}
	go(n, 0, 1)
}
```

# High order functions (HOF)
Функции высшего порядка - это функции которые принимают другие функции в качестве аргумента, либо возвращают функции в качестве возвращаемого значения или и принимают и возвращают.

Пример:
```scala
def funcA[A, B](f: A => B) = ???
def funcB[A, B, C](a: A): B => C
```

Функции высшего порядка возвращающие функции в качестве результата, можно условно разделить на несколько групп:
- обертки (оборачивают функцию доп функционалом)
- изменение поведения
- изменение самой функции

Пример обертка: 
```scala
  def logRunningTime[A, B](f: A => B): A => B = a => {
    val start = System.currentTimeMillis()
    val r = f(a)
    val end = System.currentTimeMillis()
    println(end - start)
    r
  }
```

Пример изменение поведения:
```scala
def isOdd(i: Int): Boolean = i % 2 > 0
def not[A](f: A => Boolean): A => Boolean = x => !f(x)

val isEven: Int => Boolean = not(isOdd)
```

Пример изменения функции:
```scala
def partial[A, B, C](a: A, f: (A, B) => C): B => C = b => f(a, b)
def sum(x: Int, y: Int): Int  = x + y

val p = partial(2, sum)
```

# Полиморфизм 
- параметрический
- мнимый (ad-hoc) (различные реализации интерфейса)

Полиморфизм в ООП это форма подтипов или отношений наследования

Параметрический полиморфизм когда мы хотим абстрагироваться от типа
Пример полиморфической функции
```scala
def findFirst[A](as: Array[A], p: A => Boolean): Int = {
	@annotation.tailrec
	def loop(n: Int): Int =  
		if (n >= as.length) -1 
		else if (p(as(n))) n 
		else loop(n + 1)
	loop(0)
}

findFirst(Array(7, 9, 13), (x: Int) => x == 9)
```

Проверить отсортирован ли массив
```scala
def isSorted[A](as: Array[A], ordered: (A,A) => Boolean) : Boolean = {
	@annotation.tailrec
	def loop(n:Int) : Boolean = 
		if(n >= as.length - 1) true
		else if (ordered(as(n),as(n+1))) loop(n+1)
		else false
	loop(0)
}
```
# Функциональные структуры данных
Это структуры данных методы которых не создают сайд эффектов

A* - означает что аргументов может быть переменное количество
_* - преобразует передаваемое значение в переменное количество аргументов (Vararg Splices)

Функции map и flatMap позволяют взаимодействовать с элементами внутри контейнера, не извлекая элемент.

Моделирование предметной области
- данные (иммутабельные структуры)
- вычисления (чистые функции)
- действия (функции и методы выполняющие сайд эффекты)

При моделировании предметной области важно:
- использовать общий словарь
- максимально использовать систему типов

# Pattern matching
Pattern matching - это switch на стероидах
- Типы

Пример:
   val i: Any = ???

   i match {
     case v: Int => "int"
     case v: String => "string"
     case v: List[Int] => "list int" // сюда попадем даже если v: List[String] так как дженериков нет в рантайме, они только в compile time соществуют
     case v: List[String] => "list str"
     case _ => "???" // сюда заходим всегда, если не зашли ни в один из предыдущих
   }

Использует isInstanceOf (возвращает true or false) для определения типа

Аннотация @switch оптимизирует инструкцию сопоставления с образцом в таблицу переходов (tableswitch или lookupswitch), что более оптимально по производительности в отличие от дерева решений построенном на множественных if конструкциях:
(n: @switch) match {
    case _ => "ololo"
}

Чтобы сохранить информацию о дженериках в рантайме есть конструкции TypeTags
(для этого нужно добавить зависимость "org.scala-lang" % "scala-reflect" % "2.13.3")
import scala.reflect.runtime.universe._

class Car
class Volga extends Car

def processList[T: TypeTag](json: List[T]) {
  typeOf[T] match {
    case t if t =:= typeOf[List[Int]] =>
      println("List of strings")
    case t if t =:= typeOf[List[String]] =>
      println("List of strings")
    case t if t <:< typeOf[Car] =>
      println("List of cars")
  }
}

## Структурный / структурный вложенный
case class Dog(name: String, age: Int)

    val rs =  Dog("Joe", 2) match {
      case Dog(n, a) => s"i'm dog $n"
      case _ => s"i'm someone else"
    }

case Dog(n, a) сначала проверяет тип isInstanceOf, а потом выполняет asInstanceOf то есть преобразует тип к требуемому

Деконструкцию можно сделать и так val Dog(n, age) = Dog("a", 10) тут не происходит isInstanceOf и asInstanceOf

final case class Employee(name: String, address: Address)
final case class Address(street: String, number: Int)

val alex = Employee("Alex", Address("XXX", 221))
  alex match {
    case Employee(_, Address(_, number)) => println(number)
  }


## Литералы
```scala
val dog: Animal = ???

  dog match {
    case Dog("Bim", age) =>
    case _ =>
  }

Array(7, 9, 13) // литерал массива, создает массив из 3х элементов
```

## Константы
```scala
val dog = "Bim"

  dog match {
    case "Bim" =>
    case _ =>
  }

```

## Гарды (с условием)
```scala
dog match {
      case Dog(n, a) if n == "Bim" => 
      case _ => s"i'm someone else"
    }

```

## “As” паттерн
```scala
dog match {
      case d @ Dog(_, _) => d.name
      case _ => s"i'm someone else"
    }

```

# Try
- безопасно выполнить вычисление, которое потенциально может завершиться с ошибкой
- отделить логику обработки ошибки от вычисления
- монада

sealed abstract class Try[+T]
final case class Failure[+T](exception: Throwable) extends Try[T]
final case class Success[+T](value: T) extends Try[T]

def flatMap[U](f: T => Try[U]): Try[U]
def map[U](f: T => U): Try[U]
def filter(p: T => Boolean): Try[T]
def recover[U >: T](pf: PartialFunction[Throwable, U]): Try[U]
def getOrElse[U >: T](default: => U): U
def orElse[U >: T](default: => Try[U]): Try[U]
def get: T

Пример1:
  def readFromFile(): List[Int] = {
    val s: BufferedSource = Source.fromFile(new File("ints.txt"))
    val result: List[Int] = s.getLines().map(_.toInt).toList
    s.close()
    result
  }

Пример2:
  def readFromFile(): List[Int] = {
    val s: BufferedSource = Source.fromFile(new File("ints.txt"))
    val result: List[Int] = try {
      s.getLines().map(_.toInt).toList
    } catch {
      case e =>
        println(e.getMessage)
        Nil
    } finally {
      s.close()
    }
    result
  }


Пример 3:
  def readFromFile2(): Try[List[Int]] = {
    val s: BufferedSource = Source.fromFile(new File("ints.txt"))
    val r = Try(s.getLines().map(_.toInt).toList)
    s.close()
    r
  }

  readFromFile2() match {
    case Failure(exception) =>
    println(exception.getMessage)
    case Success(value) =>
      value.foreach(println)
  }

или 

readFromFile2().foreach(l => l.foreach(println)) // он пропустить случаи Failure

Пример4:
  def readFromFile2(): Try[List[Int]] = {
    val source: Try[BufferedSource] = Try(Source.fromFile(new File("ints.txt")))

    def lines(s: BufferedSource): Try[List[Int]] =
      Try(s.getLines().map(_.toInt).toList)

    val result = for{
      s <- source
      r <- lines(s)
    } yield r
    source.foreach(_.close())
    result
  }

Пример как map отлавливает исключения:
Try("_").map(_.toInt).foreach(println)

# Многопоточность
- Процесс (создается при запуске приложения)
- Поток (существуют внутри процесса) - базовый блок конкурентности (параллельности)

Потоки есть на уровне ОС и на уровне процесса и на уровне кода

JVM поток отображается примерно 1:1 на поток на уровне ОС
При запуске Java приложения нужно минимум 1 поток (main thread), но при этом нужны еще вспомогательные потоки (например, для выполнения gc)

Для параллельного вычисления нужно минимум 2 ядра.

Thread - класс реализующий поток

public class Thread implements Runnable {
 public synchronized void start() // запуск вычислений в потоке
 public void run() // описание работы в потоке
 public final void join() throws InterruptedException // точка синхронизации внешнего потока с данным
}

Runnable - способ абстрагирования над вычислениями, которые хотим вычислить вызовом метода run

public interface Runnable {
 public abstract void run();
}

var a: Int = null.asInstanceOf[Int] // выведет 0 так как это значение по умолчанию для Int

## Управление потоками
Так как операция создания потока дорогая, то лучше использовать пул потоков и управлять им

public interface Executor {
 void execute(Runnable command);
}

Примеры:
```scala
val pool1: ExecutorService = Executors.newFixedThreadPool(2) // пул фиксирвоанного размера
val pool2: ExecutorService = Executors.newCachedThreadPool() // пул плавающего размера
val pool3: ExecutorService = Executors.newWorkStealingPool(4) // fork join pull
val pool4: ExecutorService = Executors.newSingleThreadExecutor() // пул с 1 потоком

Для неблокирующих вычислений на CPU стоит использовать FixedThreadPool и WorkStealingPool. 
Кол-во потоков = кол-во ядер + 1

Для блокирующих вычислений стоит использовать CachedThreadPool так как может создавать новые потоки при необходимости.

ExecutorService имеет метод submit с помощью которого можно получать результат
public interface ExecutorService extends Executor {
 <T> Future<T> submit(Callable<T> task);
}

public interface Callable<V> {
 V call() throws Exception;
}

public interface Future<V> {
 boolean cancel(boolean mayInterruptIfRunning);
 boolean isCancelled();
 boolean isDone();
 V get() throws InterruptedException, ExecutionException; // блокирующий
}
```

## Scala Future
Future - это контейнер, для значения которое появится в будущем. Когда мы создали такой объект, то он сразу запускает вычисления на ExecutionContext. Не монада.

trait Future[+T]
def apply[T](body: => T)(implicit executor: ExecutionContext): Future[T]

ExecutionContext - это скала враппер над ExecutorService

Скала Future дает АПИ для неблокирующего взаимодействия с результатом и для композиции. В Java есть аналог CompletableFuture.

Future

Конструкторы
    Future.successful(10)
    Future.failed(new Exception("boom"))
    Future.fromTry(Try(10))
    Future(10)(scala.concurrent.ExecutionContext.global)

scala.concurrent.ExecutionContext.global - это fork join - у него потоки являются демонами
или
scala.concurrent.ExecutionContext.Implicits.global

ExecutionContext.fromExecutorService(ec1) // создание своего ex

Await.result(cumputation, 5 seconds) // блокирующий апи

Если используем ExecutionContext, то его нужно закрывать (eс.shutdown()), все кроме fork join.

## Promise
Контейнер для некого одного значения типа T, которое может появится там в будущем
- не содержит значение типа T
- содержит значение типа T
- содержит ошибку

Future и Promise - контейнеры, но в Promise мы можем положить значение (пишем), а в Future оно появляется, мы не можем в него ничего положить (читаем).
Future - это про чтение.

Для Promise можем вернуть ассоциированный с ним Future

trait Promise[T] {
def isCompleted: Boolean // появилось ли значение в Promise
def complete(result: Try[T]):Promise[T] // как мы кладем в Promise что-то и пытаемся завершить
def tryComplete(result: Try[T]): Boolean // аналогичен complete но возвращает успех или неуспех Try
def future: Future[T] // возвращает Future связанную с Promise

# HKT
Higher-Kinded Types - тип который абстрагируется над конструктором типа `trait M[F[_]]`

```scala
def tupleF[F[_], A, B](fa: F[A], fb: F[B]): F[(A, B)] = ???

trait Bindable[F[_], A] { // HKT - тайп конструктор который в качестве тайп-параметра принимает другой тайп конструктор
 def map[B](f: A => B): F[B]
 def flatMap[B](f: A => F[B]): F[B]
}
```

`F[_]` - тайп конструктор с одним тайп-параметром, Эф с дыркой

# Implicits
## 3 группы implicit
- преобразования
- параметры
- условные

Пример преобразования:
implicit def foo(str: String): Int = ??? // conversion

или

    implicit class StringOps(str: String) {
      def trimToOption: Option[String] = Option(str).map(_.trim).filter(_.nonEmpty)
    }
//    implicit def trimToStringOps(str: String): StringOps = new StringOps(str)

    "we".trimToOption

Чем опасны:
implicit def strToInt(str: String): Int = Integer.parseInt(str)
"foo" / 42

implicit val seq = Seq("a", "b", "c")
def log(str: String) = println(str)
val res = log(2)

Пример неявного параметра:
def foo(str: String)(implicit c: C[String]): Int = ???

Пример условного:
то есть этот метод работает когда есть неявное преобразование 
implicit def foo(str: String)(implicit c: C[String]): Int = ???

## Поиск implicit
def func[T](implicit t: T)

- именованный объект, помеченный ключевым словом implicit
- тип этого объекта соответствует типу Т
- этот объект должен быть в implicit скоупе

## Скоуп по implicit
- локальный, по месту вызова
- package object
- объект компаньон для исходного типа или супер типа и для типа тайп параметра (в случае тайп-конструктора)

## Best practices
- избегайте implicit conversions
- используйте implicit classes + AnyVal для расширения методов
- view bounds deprecated
- размещайте ваши implicits в объектах компаньонах

# Тайп-классы 
- параметризованный трейт с неким функционалом, который мы хотели бы применять в широкому спектру типов
- превращение параметрического полиморфизма в ad-hoc благодаря ограничениям (bounds)
- наделяет класс определенным свойством без явного наследования

Применимы 
- когда мы не можем применить наследование из-за отсутствия доступа к исходным данным
- добавить некое свойство объекту без явного наследования

Компоненты
- тайп-конструктор
- имплисит значения или методы (инстансы тайп-класса)
- имплисит параметры (используется в тех методах где реализация завязана на тайп-классы)
- имплисит классы (опционально, для удобного синтаксиса)

Пример:
  // 1
  trait Ordering[T]{
    def less(a: T, b: T): Boolean
  }

  object Ordering{ // инстансы обычно кладут в объект-компаньон
    def from[A](f: (A, A) => Boolean): Ordering[A] = (a: A, b: A) => f(a, b) // смарт-конструктор

    // 2
    implicit val intOrdering = Ordering.from[Int]((a, b) => a < b)
  }


  // 3
  def max[A](a: A, b: A)(implicit ev: Ordering[A]): A =
    if(ev.less(a, b)) b else a

  max(10, 20)


Пример 2:
  // 1
  trait Eq[T]{
    def ===(a: T, b: T): Boolean
  }

  // 2
  object Eq{
    implicit val eqStr: Eq[String] = (a: String, b: String) => a == b
  }

  implicit class EqSyntax[T](a: T){
    // 3
    def ===(b: T)(implicit eq: Eq[T]): Boolean =
      eq.===(a, b)
  }

  "str" === ""

или другой синтаксис через контекст-баунд:

  implicit class EqSyntax[T: Eq](a: T){ // объявили контекст-баунд имплисит
    // 3
    val eq = implicitly[Eq[T]] // материализовали контекст-баунд Eq
    def ===(b: T): Boolean =
      eq.===(a, b)
  }

или через суммонер:

  // 2
  object Eq{
    def apply[T](implicit eq: Eq[T]) = eq
    
    implicit val eqStr: Eq[String] = (a: String, b: String) => a == b
  }

  implicit class EqSyntax[T: Eq](a: T){ // объявили контекст-баунд имплисит
    // 3
    def ===(b: T): Boolean =
      Eq[T].===(a, b)
  }


Пример:
  sealed trait JsValue
  object JsValue {
    final case class JsObject(get: Map[String, JsValue]) extends JsValue
    final case class JsString(get: String) extends JsValue
    final case class JsNumber(get: Double) extends JsValue
    final case object JsNull extends JsValue
  }
  
  // 1
  trait JsonWriter[T]{
    def write(v: T): JsValue
  }

  object JsonWriter{
    def from[T](f: T => JsValue): JsonWriter[T] = (v: T) => f(v) // smart-constructor

    // 2
    implicit val intJsWriter = from[Int](JsNumber(_))
    implicit val strJsWriter = from[String](JsString)

    implicit def optToJsValue[A](implicit ev: JsonWriter[A]): JsonWriter[Option[A]] =
      from[Option[A]] {
        case Some(value) => ev.write(value)
        case None => JsNull
      }
    
    def apply[T](implicit ev: JsonWriter[T]): JsonWriter[T] = ev // summoner
  }

  // 4
  implicit class JsonSyntax[T](v: T) {
    def toJson(implicit ev: JsonWriter[T]): JsValue = ev.write(v)
  }
  
  // 3
  def toJson[T: JsonWriter](v: T): JsValue = JsonWriter[T].write(v)

  toJson(10)
  toJson("abc")
  toJson[Option[Int]](Some(10))
  toJson[Option[String]](Some("abc"))

  10.toJson
  "abc".toJson
  Option(10).toJson

# Функциональный эффект

- шаблон, описание некоторого вычисления, потенциально с побочными эффектами, возможно асинхронного
- позволяет отделить описание от исполнения
- иммутабельное значение (Объект)

Зачем это?
- контроль
- композиция
- referential transparency
- тестирование

Функциональный дизайн

- иммутабельные структуры данных
- модель
- конструкторы (на основе типов конструируем искомый экземпляр)
- операторы (принимают искомый экземпляр и преобразуют его или другие типы, но + изменяют искомый экземпляр)

Модели
- исполняемая (executable encoding) // final
Конструкторы и операторы мы реализуем в терминах выполнения нашей модели

Особенности
- свободное добавление новых конструкторов и операторов
- новые сценарии выполнения влекут переработку конструкторов и операторов

- декларативная (declarative encoding) // initial
Конструкторы и операторы реализовываем, как структуры данных, на выходе получаем древовидную рекурсивную структуру

Особенности
- свободное добавление новых способов выполнения
- добавление новых примитивов потребует переработку интерпретатора

Пример:
```scala
    object executableEncoding {
      // Объявить исполняемую модель Console
      case class Console[A](run: () => A){ self =>
        def map[B](f: A => B): Console[B] = flatMap(a => Console.succeed(f(a)))
        def flatMap[B](f: A => Console[B]): Console[B] =
          Console.succeed(f(self.run()).run())
      }
      
      // Объявить конструкторы
      object Console{
        def succeed[A](a: => A): Console[A] = Console(() => a)
        def printLine(str: String): Console[Unit] = Console(() => println(str))
        def readLine(): Console[String] = Console(() => StdIn.readLine())
      }
      
      // Описать желаемую программу с помощью нашей модель
      val greet: Console[Unit] = for{
          _ <- Console.printLine("Как тебя зовут?")
          name <- Console.readLine()
          _ <- Console.printLine(s"Привет, $name")
      } yield ()
    }
```

Пример:
```scala
object declarativeEncoding {
      // Объявить декларативную модель Console
      sealed trait Console[A]{
        def map[B](f: A => B): Console[B] =
          FlatMap[A, B](this, a => Console.succeed(f(a)))
        def flatMap[B](f: A => Console[B]): Console[B] =
          FlatMap(this, f)
      }

      // Написать конструкторы
      case class PrintLine[A](str: String, rest: Console[A]) extends Console[A]
      case class ReadLine[A](f: String => Console[A]) extends Console[A]
      case class Succeed[A](str: () => A) extends Console[A]
      case class FlatMap[A, B](a: Console[A], f: A => Console[B]) extends Console[B]

      // Написать операторы
      object Console{
        def succeed[A](v: => A): Console[A] = Succeed(() => v)
        def printLine(str: String): Console[Unit] = PrintLine(str, succeed(()))
        def readLine(): Console[String] = ReadLine(str => succeed(str))
      }

      // Описать желаемую программу с помощью нашей модели
      val p1 = for{
        _ <- Console.printLine("Как тебя зовут?")
        name <- Console.readLine()
        _ <- Console.printLine(s"Привет, $name")
      } yield ()

      // Написать интерпретатор для нашей ф-циональной модели
       def interpret[A, B](console: Console[A]): A = console match {
         case PrintLine(str, rest) =>
           println(str)
           interpret(rest)
         case ReadLine(f) =>
           interpret(f(StdIn.readLine()))
         case Succeed(str) => str()
         case FlatMap(a: Console[A], f: (A => Console[B])) =>
           interpret(f(interpret(a)))
       }
    }
```

