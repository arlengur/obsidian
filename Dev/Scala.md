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

# Комментарии
```scala
// A comment!
/* Another comment */
/** A documentation comment */
```
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

## функция-метод
def sum1(x: Int, y: Int): Int = x + y
- именованные параметры
- конвертируются в функции-значения

val sum4: (Int, Int) => Int = sum1 // компилятор преобразует функцию-метод sum1 в функцию-значение
val sum5 = sum1 _

## функция-значение
val foo: Int => Int
 val sum2: (Int, Int) => Int = (a, b) => a + b
 val sum2: (Int, Int) => Int = (_: Int) + (_: Int)
 val sum2: (Int, Int) => Int = _ + _
- являются объектами
- можно сохранять в переменные
- можно передавать на вход другим функциям и возвращать в качестве значения

sum2(2, 3) эквивалентно вызову функции apply на `Function2[Int, Int, Int]` а `(a, b) => a + b` является созданием экземпляра объекта `Function2[Int, Int, Int]`

Функцию-значение sum2 (так как является объектом) можно присвоить в какое-то значение:
val sum3 = sum2

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

## Currying
процесс преобразования функции от нескольких аргументов в несколько функций от одного аргумента
def foo(x: Int, y: Int): Int
val foo: Int => Int => Int

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

sealed гарантирует что все наследники находятся в том же файле и это полезно при паттерн матчинге.

# List
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

## Проблемы List
- Занимает в два раза больше массива
- Много элементов - нагрузка на сборщик мусора
- Время выполнения многих операций пропорционально длине O(N)
- Вставка в конец - только с полным копированием (список раскручивается в стек и потом стек собирается в новый список)
- средства диагностики heap dump плохо работают

## Методы

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
## Задачки
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
Pure functions - функции, которые для одних и тех же аргументов вычисляют одни и те же результаты и не делают никаких сайд эффектов, то есть функция не делает никаких сайд эффектов кроме как вычисление результата в зависимости от переданного на вход значения

f: A => B - функция, которой на вход подают тип А, а на выходе получаем тип В - выполняет вычисление которое связывает каждое значение а типа А только с одним значением b типа B. Пример: intToString: Int => String

def sum(x: Int, y: Int): Int = x + y - чистая функция

Чистые функции позволяют делать кеширование и код становится более понятным.

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
  def factRec(n: Int): Int =
    if(n <= 0) 1 else n * factRec(n - 1)

  def factRecTail(n: Int): Int = {
    @tailrec
    def loop(i: Int, acc: Int): Int = {
      if(i <= 0) acc
      else loop(i - 1, i * acc)
    }
    loop(n, 1)
  }

Пример Фибоначчи:
F0 = 0, F1 = 1, Fn = Fn-1 + Fn - 2
0, 1, 1, 2, 3, 5

   def fibRec(n: Int): Int = {
    if(n==0) 0
    else if(n==1) 1
    else fib(n-1)+fib(n-2)
   }

   def fibREcTail(n: Int): Int = {
    def go(n: Int, acc: Int, x: Int): Int = n match {
      case 0 => acc
      case _ => go(n-1, x, acc+x)
    }
    go(n, 0, 1)
   }

# High order functions (HOF)
Функции высшего порядка - это функции которые принимают другие функции в качестве аргумента, либо возвращают функции в качестве возвращаемого значения или и принимают и возвращают.

Пример:
```
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

Полиморфизм 
- параметрический (метод работающий для разных типов, def max(a: A, b: A): A = ???) 
- мнимый (ad-hoc) (различные реализации интерфейса)

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

