Это интерфейс или API, который представляет функциональность, которую мы хотим реализовать. В скала представлены трейтом с, как минимум, одним параметром.

Универсальный импорт
- `import cats._` - импорт всех тайп классов
- `import cats.implicits._` - импорт всех стандартных экземпляров тайп классов и интерфейсов syntax для них
```scala
// Define a very simple JSON AST
sealed trait Json  
final case class JsObject(get: Map[String, Json]) extends Json 
final case class JsString(get: String) extends Json  
final case class JsNumber(get: Double) extends Json  
final case object JsNull extends Json

// The "serialize to JSON" behaviour is encoded in this trait
trait JsonWriter[A] {
  def write(value: A): Json
}
```

## Экземпляры тайп класса
```scala
final case class Person(name: String, email: String)

object JsonWriterInstances {
  implicit val stringWriter: JsonWriter[String] =
    new JsonWriter[String] {
      def write(value: String): Json =
        JsString(value)
    }

  implicit val personWriter: JsonWriter[Person] =
    new JsonWriter[Person] {
      def write(value: Person): Json =
        JsObject(Map(
          "name" -> JsString(value.name),
          "email" -> JsString(value.email)
        ))
	}
  // etc...
}
```

## Интерфейс объектов
```scala
object Json {  
	def toJson[A](value: A)(implicit w: JsonWriter[A]): Json =
	    w.write(value)
}
```

Чтобы использовать этот объект нужно импортировать экземпляр тайп класса и вызвать необходимый метод

```scala
import JsonWriterInstances._

Json.toJson(Person("Dave", "dave@example.com"))
// JsObject(Map(name -> JsString(Dave), email -> JsString(dave@example.com)))
```

## Интерфейс Syntax
Мы можем использовать методы расширения чтобы расширить существующие типы методами интерфейса

```scala
object JsonSyntax {
  implicit class JsonWriterOps[A](value: A) {
    def toJson(implicit w: JsonWriter[A]): Json =
      w.write(value)
  } 
}
```

Для использования интерфейса Syntax его необходимо импортировать весте с необходимыми нам экземплярами типов
```scala
import JsonWriterInstances._
import JsonSyntax._

Person("Dave", "dave@example.com").toJson
// JsObject(Map(name -> JsString(Dave), email -> JsString(dave@example.com)))
```

 Метод implicitly
 Стандартная библиотека Scala предоставляет общий интерфейс тайп классов - implicitly:
 ```scala
 def implicitly[A](implicit value: A): A = value
```

Мы можем использовать implicitly чтобы вызвать любое значение из имплисит области видимости. Мы предоставляем тип который нам нужен и implicitly делает все остальное:
```scala
import JsonWriterInstances._
implicitly[JsonWriter[String]]
```

## Имплисит область видимости  (скоуп)
То где компилятор ищет кандидатов на экземпляры для тайп класса

Имплисит скоуп состоит из
- локальные или унаследованные определения
- импортированные определения
- определения в объекте-компаньоне тайп класса или в параметре типа тайп класса (например в JsonWriter или String)

Определения включаются в имплисит скоуп, если они помечены специальным словом implicit. Если компилятор видит несколько определений кандидатов, то мы получим ambiguous implicit values ошибки.

Где располагать экземпляры имплиситов
- в объектах типа JsonWriterInstances 
	- импортируем экземпляры получает при помощи импорта
- в трейтах 
	- импортируем экземпляры получает при помощи наследования
- в компаньон-объекте тайп класса 
	- не нужно импортировать тк все у же в имплисит скоупе
- в компаньон-объекте типа тайп класса 
	- не нужно импортировать тк все у же в имплисит скоупе

Рекурсивные решение для имплиситов
Если  нам надо написать экземпляр для Option(A), то сделать это можно так
```scala
implicit def optionWriter[A]  
(implicit writer: JsonWriter[A]): JsonWriter[Option[A]] =
  new JsonWriter[Option[A]] {
    def write(option: Option[A]): Json =
      option match {
        case Some(aValue) => writer.write(aValue)
        case None         => JsNull
	} 
  }
```
метод конструирует JsonWriter for Option(A) полагаясь на то что JsonWriter(A) существует. 
Чтобы такая рекурсия работала важно чтобы параметр был implicit. 
implicit методы без implicit параметра являются другим scala-паттерном - неявные преобразования.
Когда компилятор видит выражение:
```scala
Json.toJson(Option("A string"))
```

он находит `JsonWriter[Option[A]]` и затем рекурсивно `JsonWriter[String]`

В Cats тайп классы определены в пакете cats, объект-компаньон содержит метод apply, который находит экземпляр типа, который мы определили, он использует implicits, чтобы найти отдельные экземпляры, поэтому их нужно ипортировать в скоуп
# Show
```scala
package cats

trait Show[A] {
  def show(value: A): String
}

import cats.instances.int._
val showInt = Show.apply[Int]
val intAsString: String = showInt.show(123) // "123"

// через интерфейс syntax
import cats.syntax.show._
val shownInt = 123.show // "123"
```

Импортирование экземпляров по умолчанию
- cats.instances.string
- cats.instances.list
- cats.instances.option
- cats.instances.all - импортирует все доступные по умолчанию экземпляры

Определение своих экземпляров
```scala
import cats.syntax.show._
import java.util.Date

implicit val dateShow: Show[Date] =
  new Show[Date] {
    def show(date: Date): String =
      s"${date.getTime}ms since the epoch."
}

new Date().show // 1708610198534ms since the epoch.
```

Чтобы облегчить процесс создания пользовательских экземпляров есть 2 метода
```scala
// преобразует функцию в экземпляр Show
def show[A](f: A => String): Show[A] = ???

// Создает экземпляр Show из метода 'toString'
def fromToString[A]: Show[A] = ???
```

Пример
```scala
implicit val dateShow: Show[Date] = 
Show.show(date => s"${date.getTime}ms since the epoch.")

Show.fromToString[Date].show(new Date())
```

# Eq
Определяет типо безопасное равенство между экземплярами типов

```scala
package cats

trait Eq[A] {
  def eqv(a: A, b: A): Boolean
}
```

Интерфейс syntax определяет 2 метода сравнения
- === - сравнивает 2 объекта на равенство
- =!= - сравнивает 2 объекта на неравенство

```scala
import cats.Eq  
val eqInt = Eq[Int]  
eqInt.eqv(123, 123) // true
eqInt.eqv(123, "234") // type mismatch

// через syntax
import cats.syntax.eq._ // for === and =!=  
123 === 123 // true
(Some(1): Option[Int]) === (None: Option[Int])
Option(1) === Option.empty[Int]

import cats.syntax.option._  
import cats.syntax.eq._  
1.some === none[Int]
```

Определение польхзовательского типа
```scala
import cats.Eq  
import cats.syntax.eq._  
  
import java.util.Date  
implicit val dateEq: Eq[Date] = Eq.instance[Date] { (date1, date2) =>  
	date1.getTime === date2.getTime  
}  
  
val x = new Date() // now  
val y = new Date() // a bit later than now  
  
x === x
x === y
```

# Ковариантность
Если А подтип В, то `List[A]` подтип `List[B]`
Позволяет заменять коллекции одного типа коллекциями его под типа. Везде где нам нужна `List[Shape]` можно использовать `List[Circle]` так как Circle подтип Shape.
Используется когда нам нужно получить что-то из контейнера

# Контрвариантность
Если А подтип В, то `List[B]` подтип `List[A]`
Используется когда нам нужно получить что-то на вход
```scala
sealed trait Shape
case class Circle(radius: Double) extends Shape

trait JsonWriter[-A] {
  def write(value: A): Json
}

val shape: Shape = ???
val circle: Circle = ???

val shapeWriter: JsonWriter[Shape] = ???
val circleWriter: JsonWriter[Circle] = ???

def format[A](value: A, writer: JsonWriter[A]): Json = writer.write(value)
```

Мы можем передать circle с любым Writer так как все Circle являются Shape, но shape мы не можем передать с circleWriter так как не все Shape являются Circle.

# Инвариантность
Вне зависимости от отношений между А и В типы `F[A]` и `F[B]` не являются подтипами друг друга 

Когда компилятор ищет имплиситы он смотрит на соответствие типов или на то является ли один тип под типом другого.

| Вариантность тайп класса | Инвариантный | Ковариантный | Контрвариантный |
| ---- | ---- | ---- | ---- |
| Используется экземпляр супер типа | нет | нет | да |
| Используется экземпляр подтипа | нет | да | нет |
# Моноиды и Полугруппы

## Моноид для типа А это
- операция комбинирования с типом (A, A) => A
- нулевой элемент типа А

```scala
trait Monoid[A] {
  def combine(x: A, y: A): A
  def empty: A
}
```

## Законы
Для моноида должны выполняться законы:
- ассоциативности
	- (x + y) + z == x + (y + z)
- идентичности
	- a + 0 == 0 + a == a

```scala
def associativeLaw[A](x: A, y: A, z: A)
(implicit m: Monoid[A]): Boolean = {
  m.combine(x, m.combine(y, z)) == m.combine(m.combine(x, y), z)
}

def identityLaw[A](x: A)
      (implicit m: Monoid[A]): Boolean = {
  (m.combine(x, m.empty) == x) && (m.combine(m.empty, x) == x)
}
```

Пример моноидов:
- сложение int
- умножение int
- конкатенация строк

Пример не моноида:
- вычитание int

## Полугруппа для типа А это
- операция комбинирования с типом (A, A) => A

```scala
trait Semigroup[A] {
  def combine(x: A, y: A): A
}
```

Пример полугруппы:
- NonEmptyList - полугруппа, но не моноид

Полугруппа являлется подмножеством моноидов, поэтому моноид можно определить так
```scala
trait Monoid[A] extends Semigroup[A] {
  def empty: A
}
```
Объект-компаньон имеет метод apply, который возвращает экземпляр тайп-класса, для конкретного типа:
```scala
import cats.Monoid // alias cats.kernel.Monoid
import cats.Semigroup // alias cats.kernel.Semigroup

Monoid.apply[String].combine("Hi ", "there")
// or
Monoid[String].combine("Hi ", "there")
Monoid[Option[Int]].combine(Option(1), Option(2))

Monoid.apply[String].empty
// or
Monoid[String].empty

Semigroup[String].combine("Hi ", "there")
```

Экземпляры тап-классов для моноидов и полугрупп определены в cats.instances._

Интерфейс syntax для полугруппы
```scala
import cats.syntax.semigroup._  
   
"Hi " |+| "there" |+| Monoid[String].empty
```

Пример метода для любого типа являющегося моноидом:
```scala
def addAll[A](values: List[A])(implicit monoid: Monoid[A]): A =
  values.foldRight(monoid.empty)(_ |+| _)

addAll(List(1, 2, 3)) // 6
addAll(List(None, Some(1), Some(2))) // Some(3)
```

# Функтор
map - это способ упорядочить вычисления на значениях контейнера, игнорируя сложности продиктованные определенным типом данных

Future - это функтор который упорядочивает асинхронные вычисления, выстраивая их в очередь и применяя их как только Future будет выполнена. Функтор применяется, как только Future завершится, если  он еще не завершился, то он будет применен позже.

Функция с одним аргументом - это функтор

```scala
import cats.syntax.functor._ // for map  
  
val func1: Int => Double = (x: Int) => x.toDouble  
val func2: Double => Double = (y: Double) => y * 2  
(func1 map func2)(1) // composition using map  
(func1 andThen func2)(1) // composition using andThen  
func2(func1(1)) // composition written out by hand
```
Когда мы используем операцию map мы добавляет новую операцию к цепочке вычислений, но вычисления будут произведены только тогда, когда мы передадим параметр в полученную функцию.

Функтор - это [конструктор типа](#Конструкторы%20типов) `F[_]` с операцией map: `(A => B) => F[B]`

```scala
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}
```

## Законы
Для функтора должны выполняться законы:
- идентичности - map на идентичной функции ничего не делает
	- fa.map(a => a) == fa
- композиции - map с функциями f и g делает сначала map(f) а потом map(g)
	- fa.map(g(f(_))) == fa.map(f).map(g)

Пример
```scala
import cats.Functor  
val list1 = List(1, 2, 3)  
Functor[List].map(list1)(_ * 2) // List(2, 4, 6)  
  
val option1 = Option(123)  
Functor[Option].map(option1)(_.toString) // Some(123)
```

Functor имеет метод lift, который преобразует функцию A => B в `F[A] => F[B]`
```scala
import cats.Functor  
  
val func = (x: Int) => x + 1 // Int => Int  
val liftedFunc = Functor[Option].lift(func) // Option[Int] => Option[Int]
liftedFunc(Option(5)) // Some(6)
```

Functor имеет метод as, который заменяет все значения внутри контейнера на переданное
```scala
val list1 = List(1, 2, 3)  
Functor[List].as(list1, "As") // List(As, As, As)
```

Интерфейс syntax для функтора

Пример
```scala
import cats.syntax.functor._  
  
val func1 = (a: Int) => a + 1  
val func2 = (a: Int) => a * 2  
val func3 = (a: Int) => s"${a}!"  
val func4 = func1 map func2 map func3
func4(123) // 248!
List(1, 2, 3).as("as") // List(as, as, as)
```

Пример
```scala
import cats.Functor  
import cats.syntax.functor._  
  
def doMath[F[_]](start: F[Int])(implicit functor: Functor[F]): F[Int] = start.map(n => n + 1 * 2)  
  
doMath(Option(20)) // Some(22)
doMath(List(1, 2, 3)) // List(3, 4, 5)
```

Пользовательские экземпляры
```scala
implicit val optionFunctor: Functor[Option] =  
	new Functor[Option] {  
		def map[A, B](value: Option[A])(func: A => B): Option[B] = value.map(func)  
	}  
  
import scala.concurrent.{ExecutionContext, Future}  
implicit def futureFunctor(implicit ec: ExecutionContext): Functor[Future] =  
	new Functor[Future] {  
		def map[A, B](value: Future[A])(func: A => B): Future[B] = value.map(func)  
	}
```

## Контрвариантный функтор

Реализует метод contramap добавляющий операцию к цепочке вычислений. Используется в типах данных выполняющих преобразования

Пример
```scala
trait Printable[A] { self =>  
	def format(value: A): String  
	  
	def contramap[B](func: B => A): Printable[B] =  
		new Printable[B] {  
			def format(value: B): String = self.format(func(value))  
		}  
	  
	def format[A](value: A)(implicit p: Printable[A]): String = p.format(value)  
}

implicit val stringPrintable: Printable[String] =  
	new Printable[String] {  
		def format(value: String): String =  
			s"'${value}'"  
	}  
implicit val booleanPrintable: Printable[Boolean] = 
	new Printable[Boolean] {  
		def format(value: Boolean): String =  
			if (value) "yes" else "no"  
	}  
  
stringPrintable.format("hello") // 'hello'  
booleanPrintable.format(true) // yes

final case class Box[A](value: A)  
  
implicit def boxPrintable[A](implicit p: Printable[A]): Printable[Box[A]] =  
	new Printable[Box[A]] {  
		def format(box: Box[A]): String = p.format(box.value)  
	}

// более простой вариант
implicit def boxPrintable[A](implicit p: Printable[A]): Printable[Box[A]] = p.contramap[Box[A]](_.value)

boxPrintable[Box[String]].format(Box("hello")) // 'hello'  
boxPrintable[Box[Boolean]].format(Box(true)) // yes
```

## Инвариантный функтор

Реализует метод  imap который эквивалентен методам map и contramap. Принцип его работы подобен кодированию и декодированию данных

```scala
trait Codec[A] { self =>  
	def encode(value: A): String  
	def decode(value: String): A  
	  
	def imap[B](dec: A => B, enc: B => A): Codec[B] = new Codec[B] {  
		def encode(value: B): String = self.encode(enc(value))  
		def decode(value: String): B = dec(self.decode(value))  
	}  
}  
  
def encode[A](value: A)(implicit c: Codec[A]): String = c.encode(value)  
def decode[A](value: String)(implicit c: Codec[A]): A = c.decode(value)  
  
implicit val stringCodec: Codec[String] =  
	new Codec[String] {  
		def encode(value: String): String = value  
		def decode(value: String): String = value  
	}  
  
implicit val intCodec: Codec[Int] = stringCodec.imap(_.toInt, _.toString)  
implicit val booleanCodec: Codec[Boolean] = stringCodec.imap(_.toBoolean, _.toString)  

encode(123) // 123
decode[Int]("123") // 123
  
final case class Box[A](value: A)  
  
implicit def boxCodec[A](implicit c: Codec[A]): Codec[Box[A]] = c.imap[Box[A]](Box(_), _.value)  

encode(Box(123)) // 123
decode[Box[Int]]("123") // 123
```

## Контрвариантность в Cats
```scala
trait Contravariant[F[_]] {
  def contramap[A, B](fa: F[A])(f: B => A): F[B]
}
```

Пример
```scala
import cats.{Contravariant, Show}  

val showString = Show[String]  
val showSymbol = Contravariant[Show].contramap(showString)((sym: Symbol) => s"'${sym.name}")  
showSymbol.show(Symbol("dave"))
```

Через syntax
```scala
import cats.Show  
import cats.syntax.contravariant._  
  
val showString = Show[String]  
showString  
.contramap[Symbol](sym => s"'${sym.name}")  
.show(Symbol("dave"))
```

## Инвариантность в Cats
```scala
trait Invariant[F[_]] {  
  def imap[A, B](fa: F[A])(f: A => B)(g: B => A): F[B]
}
```

В Cats нет полугруппы для Symbol, но есть для String. 
Сделаем функцию которая принимает 2 Symbol, преобразует их в строки, соединяет методом combine и преобразует обратно в Symbol.

Пример создания полугруппы для Symbol
```scala
import cats.syntax.semigroup._  
import cats.syntax.invariant._  
import cats.Semigroup  
  
implicit val symbolSemigroup: Semigroup[Symbol] = Semigroup[String].imap(Symbol.apply)(_.name)  
  
Symbol("a") |+| Symbol("few") |+| Symbol("words")
```






# Конструкторы типов

List - конструктор типа, принимает 1 параметр
`List[A]` - тип, полученный применением параметра типа

Это подобно тому как функции являются конструкторами значений
math.abs - функция принимающая 1 параметр
math.abs(x) - значение, полученное применением параметра значения

Конструкторы объявляются при помощи `_`, он определяет как много дыр в конструкторе типа, но при использовании конструктора типа мы ссылаемся на него просто по имени
```scala
// объявляем F при помощи нижнего подчеркивания
def myMethod[F[_]] = {
  // ссылаемся на F без нижнего подчеркивания
  val functor = Functor.apply[F]
}
```

В функциях мы делаем аналогично, при определении параметра мы указываем его тип, а при использовании просто ссылаемся по имени
```scala
// определение f указывая параметр типа
def f(x: Int): Int =  
	// ссылаемся на x без типа 
	x*2
```

# Функторы и вариантность
В является подтипом А, если существует функция В => A. 
Аналогично этому 
- пусть F - ковариантный функтор, если у нас есть `F[B]` и функция В => A, тогда мы можем получить `F[A]`
- пусть F - контрвариантный функтор, если у нас есть `F[A]` и функция В => A, тогда мы можем получить `F[B]`
- пусть F - инвариантный функтор, если у нас есть `F[A]` и функции A => B и B => A, тогда мы можем получить `F[В]`

# Монады
Механизм последовательных вычислений, имеющий две операции
- pure - возводит значение в контекст
- flatMap - обеспечивает последовательное выполнение шагов: берет значение из контекста и создает новый контекст в последовательности: `(F[A], A => F[B]) => F[B]`

```scala
trait Monad[F[_]] {
  def pure[A](value: A): F[A]
  def flatMap[A, B](value: F[A])(func: A => F[B]): F[B]
}
```

Примеры: Option, List,  Future

## Законы
Монада должна удовлетворять следующим законам
- Левая идентичность 
	- `pure(a).flatMap(func) == func(a)`
- Правая идентичность
	- `m.flatMap(pure) == m`
- Ассоциативность
	- `m.flatMap(f).flatMap(g) == m.flatMap(x => f(x).flatMap(g))`

Каждая монада является функтором, напишем map для монады
```scala
trait Monad[F[_]] {
  def pure[A](a: A): F[A]
  def flatMap[A, B](value: F[A])(func: A => F[B]): F[B]
  def map[A, B](value: F[A])(func: A => B): F[B] = flatMap(value)(a => pure(func(a)))
}
```

Примеры
```scala
import cats.Monad

val opt1 = Monad[Option].pure(3) // Some(3)  
val opt2 = Monad[Option].flatMap(opt1)(a => Some(a + 2)) // Some(5)  
val opt3 = Monad[Option].map(opt2)(a => 100 * a) // Some(500)

val list1 = Monad[List].pure(3) // List(3)  
val list2 = Monad[List].flatMap(List(1, 2, 3))(a => List(a, a * 10)) // List(1, 10, 2, 20, 3, 30)  
val list3 = Monad[List].map(list2)(a => a + 123) // List(124, 133, 125, 143, 126, 153)

import scala.concurrent.Future  
import scala.concurrent.ExecutionContext.Implicits.global  
import scala.concurrent.Await  
import scala.concurrent.duration.DurationInt
  
val fm = Monad[Future].flatMap(Monad[Future].pure(1))(x => Monad[Future].pure(x + 2)) // Future(Success(3))
Await.result(fm, 1.second) // 3
```

## Интерфейс syntax
```scala
import cats.syntax.applicative._  
  
1.pure[Option] // Some(1)  
1.pure[List] // List(1)

import cats.Monad  
import cats.syntax.functor._ // for map  
import cats.syntax.flatMap._ // for flatMap  
  
def sumSquare[F[_]: Monad](a: F[Int], b: F[Int]): F[Int] = a.flatMap(x => b.map(y => x*x + y*y))  
  
sumSquare(Option(3), Option(4)) // Some(25)  
sumSquare(List(1, 2, 3), List(4, 5)) // List(17, 26, 20, 29, 25, 34)
```

можно записать sumSquare через for comprehension:
```scala
def sumSquare[F[_] : Monad](a: F[Int], b: F[Int]): F[Int] =  
	for {  
		x <- a  
		y <- b  
	} yield x * x + y * y
```

## Монада Id
Это тайп алиас который оборачивает простые значения в тайп-конструктор от одного параметра
```scala
type Id[A] = A

"Dave": Id[String]
123: Id[Int]
List(1, 2, 3): Id[List[Int]]
```

Теперь можно вызвать sumSquare для простых значений (Scala не может может сопоставить типы и конструкторы типов когда ищет имплиситы, поэтому необходимо явно указать тип)
```scala
sumSquare(3: Id[Int], 4: Id[Int]) // 25
```

Cats обеспечивает тайп-классы для Id, включая Functor и Monad
```scala
import cats.Monad  
import cats.syntax.functor._ // for map  
import cats.syntax.flatMap._ // for flatMap  
  
val a = Monad[Id].pure(3) // 3  
val b = Monad[Id].flatMap(a)(_ + 1) // 4  
  
for {  
	x <- a  
	y <- b  
} yield x + y // 7
```

Возможность абстрагироваться над монадическим и немонадическим кодом дает большие возможности, когда например в продакшине мы используем Future а в тестах Id

Методы pure, map, и flatMap для Id
```scala
import cats.Id  

// A и Id[A] одно и тоже, поэтому
def pure[A](value: A): Id[A] = value 
def map[A, B](initial: Id[A])(func: A => B): Id[B] = func(initial)  
def flatMap[A, B](initial: Id[A])(func: A => Id[B]): Id[B] = func(initial)
```

## Монада Either

```scala
import cats.syntax.either._ // for asRight  
  
val a = 3.asRight[String] // Right(3)  
val b = 4.asRight[String] // Right(4)  
  
for {  
	x <- a  
	y <- b  
} yield x * x + y * y
```

Умный конструктор asRight имеет тип возращаемого значения Either, вместо стандартных Left.apply и Right.apply и позволяет определить тип с только одним тайп-параметром
```scala
import cats.syntax.either._  
  
def countPositive(nums: List[Int]) =  
	nums.foldLeft(0.asRight[String]) { (accumulator, num) =>  
		if (num > 0) {  
			accumulator.map(_ + 1)  
		} else {  
			Left("Negative. Stopping!")  
		}  
	}  
  
countPositive(List(1, 2)) // Right(2)
```

Также Cats добавила в Either два полезных метода catchOnly и catchNonFatal
```scala
  
import cats.syntax.either._  
  
Either.catchOnly[NumberFormatException]("foo".toInt) // Left(java.lang.NumberFormatException: For input string: "foo")  
Either.catchNonFatal(sys.error("Badness") // Left(java.lang.RuntimeException: Badness)
```

Создание Either из других типов
```scala
Either.fromTry(scala.util.Try("foo".toInt))
Either.fromOption[String, Int](None, "Badness")
```

Преобразование Either
```scala
"Error".asLeft[Int].getOrElse(0) // 0
"Error".asLeft[Int].orElse(2.asRight[String]) // Right(2)

// ensure проверяет удовлетворяет ли значение в Right условию
-1.asRight[String].ensure("Must be non-negative!")(_ > 0) // Left(Must be non-negative!)

"error".asLeft[Int].recover {  
	case _: String => -1  
} // Right(-1)
  
"error".asLeft[Int].recoverWith {  
	case _: String => Right(-1)  
} // Right(-1)

"foo".asLeft[Int].leftMap(_.reverse) // Left(oof)
6.asRight[String].bimap(_.reverse, _ * 7) // Right(42)
"bar".asLeft[Int].bimap(_.reverse, _ * 7) // Left(rab)

// меняет местами Left и Right
123.asRight[String].swap // Left(123)
```

Обработка ошибок

Можно создать ADT для разных вариантов возможных ошибок
```scala
import cats.syntax.either._  
  
object wrapper {  
	sealed trait LoginError extends Product with Serializable  
	  
	final case class UserNotFound(username: String) extends LoginError 
	final case class PasswordIncorrect(username: String) extends LoginError  
	case object UnexpectedError extends LoginError  
};  
  
import wrapper._  
  
case class User(username: String, password: String)  
  
def handleError(error: LoginError): Unit =  
	error match {  
		case UserNotFound(u) =>  
			println(s"User not found: $u")  
		case PasswordIncorrect(u) =>  
			println(s"Password incorrect: $u")  
		case UnexpectedError =>  
			println(s"Unexpected error")  
	}  
  
User("dave", "passw0rd").asRight.fold(handleError, println)  
UserNotFound("dave").asLeft.fold(handleError, println)
```

## Монада MonadError

```scala
trait MonadError[F[_], E] extends Monad[F] {
	// поднимает ошибку в контекст
	def raiseError[A](e: E): F[A]
	// обработка ошибки
	def handleErrorWith[A](fa: F[A])(f: E => F[A]): F[A]
	// обработка всех ошибок
	def handleError[A](fa: F[A])(f: E => A): F[A]

	// Ошибка, если значение в контексте не удовлетворяет условию 
	def ensure[A](fa: F[A])(e: E)(f: A => Boolean): F[A]
}
```

Определяется двумя тайп-параметрами
- F - тип монады
- E - тип ошибки в F

```scala
import cats.MonadError  
  
type ErrorOr[A] = Either[String, A]  
val monadError = MonadError[ErrorOr, String]

val success = monadError.pure(42) // Right(42)
val failure = monadError.raiseError("Badness") // Left(Badness)

monadError.handleError[Int](failure) {  
	case "Badness" => 42  
	case _ => -1  
} // Right(42)

monadError.handleErrorWith(failure) {  
	case "Badness" =>  
		monadError.pure("It's ok")  
	case _ =>  
		monadError.raiseError("It's not ok")  
} // Right(It's ok)

monadError.ensure(success)("Number too low!")(_ > 1000) // Left(Number too low!)
```

Интерфейс
```scala
import cats.syntax.applicative._ // for pure  
import cats.syntax.applicativeError._ // for raiseError etc  
import cats.syntax.monadError._ // for ensure  
  
type ErrorOr[A] = Either[String, A]  
  
val success = 42.pure[ErrorOr]  
val failure = "Badness".raiseError[ErrorOr, Int]  
  
failure.handleErrorWith {  
	case "Badness" => 256.pure[ErrorOr]  
	case _ => "It's not ok".raiseError  
}  
success.ensure("Number to low!")(_ > 1000)
```

Экземпляры
```scala
import scala.util.Try  
import cats.syntax.applicativeError._  
  
val exn: Throwable = new RuntimeException("It's all gone wrong")  
exn.raiseError[Try, Int] // Failure(java.lang.RuntimeException: It's all gone wrong)
```

Пример
```scala
import cats.MonadError  
import cats.syntax.applicative._  
import cats.syntax.applicativeError._  
  
import scala.util.Try  
  
  
def validateAdult[F[_]](age: Int)(implicit me: MonadError[F, Throwable]): F[Int] =  
	if (age >= 18) age.pure[F]  
	else new IllegalArgumentException("Age must be greater than or equal to 18").raiseError[F, Int]  
  
validateAdult[Try](18) // Success(18)
  
type ExceptionOr[A] = Either[Throwable, A]  
validateAdult[ExceptionOr](-1) // Left(java.lang.IllegalArgumentException: Age must be greater than or equal to 18)
```

## Монада Eval
Позволяет выполнять вычисления по необходимости и реализует стекобезопасные вычисления 

Вычисления call‐by‐value
- производятся в месте определения
- результат вычислений запоминается

```scala
val x = {
  println("Computing X")
  math.random
}
```

Вычисления call‐by‐name
- производятся в момент вызова
- результат вычислений не запоминается

```scala
def y = {
  println("Computing Y")
  math.random
}
```

Вычисления call‐by‐need
- производятся в момент вызова
- результат вычислений запоминается
```scala
lazy val z = {
  println("Computing Z")
  math.random
}
```

Eval имеет 3 типа:
- Now (call‐by‐value)
- Always (call‐by‐name)
- Later (call‐by‐need)

```scala
import cats.Eval  

val now = Eval.now(math.random + 1000) // Now(1000.2876728707711)  
now.value // 1000.5530571543644  
val always = Eval.always(math.random + 3000) // cats.Always@6385cb26  
always.value // 3000.3810001352454  
val later = Eval.later(math.random + 2000) // cats.Later@38364841  
later.value // 2000.12667838288
```

Как и все монады Eval позволяет строить цепочки вычислений, вычисления хранятся как список функций, которые вычисляются в момент вызова метода value

```scala
val greeting = Eval  
.always { println("Step 1"); "Hello" }  
.map { str => println("Step 2"); s"$str world" }

greeting.value
// Step 1
// Step 2
// Hello world
```

```scala
val ans = for {  
	a <- Eval.now {  
		println("Calculating A");  
		40  
	}  
	b <- Eval.always {  
		println("Calculating B");  
		2  
	}  
} yield {  
	println("Adding A and B")  
	a + b  
}
// Calculating A

ans.value
// Calculating B
// Adding A and B
// 42
```

Eval имеет метод memoize он позволяет кэшировать все что было вызвано до него и не кэшировать все что идет поле него

```scala
import cats.Eval  
  
val saying = Eval  
	.always {  
	println("Step 1"); "The cat"  
	}  
	.map { str => println("Step 2"); s"$str sat on" }  
	.memoize  
	.map { str => println("Step 3"); s"$str the mat" }

saying.value
// Step 1
// Step 2
// Step 3
// The cat sat on the mat

saying.value
// Step 3
// The cat sat on the mat
```

Трамполирование и Eval.defer
Методы map и flatMap трамполированы это позволяет делать вложенные вызовы внутри них без создания новых окон в стеке то есть они стекобезопасные

```scala
def factorial(n: BigInt): BigInt =  
	if (n == 1) n else n * factorial(n - 1)
factorial(50000) // StackOverflowError

import cats.Eval

def factorial(n: BigInt): Eval[BigInt] =  
	if (n == 1) {  
		Eval.now(n)  
	} else {  
		Eval.defer(factorial(n - 1) map (_ * n))  
	}  
  
factorial(50000).value
```

Eval.defer - использует существующий экземпляр Eval и откладывает его вычисление. Он тоже трамполирован.
Транполирование - это не бесплатное удобство оно переносит создание цепочки объектов из стека в heap, поэтому длина цепочки зависит от объема heap'a.

Пример
```scala
def foldRight[A, B](as: List[A], acc: B)(fn: (A, B) => B): B = 
	as match {
	    case head :: tail =>
	      fn(head, foldRight(tail, acc)(fn))
	    case Nil =>
	      acc
	}

foldRight((1 to 100000).toList, 0L)(_ + _) // StackOverflowError

// перепишем foldRight, щаменив в нем все В на Eval[B] и рекурсивный вызов обернем в Eval.defer
def foldRightEval[A, B](as: List[A], acc: Eval[B])(fn: (A, Eval[B]) => Eval[B]): Eval[B] =  
	as match {  
		case head :: tail =>  
			Eval.defer(fn(head, foldRightEval(tail, acc)(fn)))  
	case Nil =>  
		acc  
	}

// тогда
def foldRight[A, B](as: List[A], acc: B)(fn: (A, B) => B): B = 
	foldRightEval(as, Eval.now(acc)) { (a, b) =>  
		b.map(fn(a, _))  
	}.value

foldRight((1 to 100000).toList, 0L)(_ + _) // 5000050000
```

## Монада Writer
Позволяет хранить лог (сообщения, ошибки или дополнительные данные) вместе с вычислениями и получить их вместе с конечным результатом. Часто применяется в многопоточном программировании, получая результаты из разных потоков вместе с сообщениями мы избегаем варианта когда можно перепутать результаты.
`Writer[W, A]`содержит 2 параметра log типа W и результат типа A.

```scala
import cats.data.Writer  

Writer(Vector(
	"It was the best of times", 
	"it was the worst of times"
), 1859) // WriterT((Vector(It was the best of times, it was the worst of times),1859))

// через syntax
import cats.syntax.writer._   
val a = 123.writer(Vector("msg1", "msg2", "msg3")) // WriterT((Vector(msg1, msg2, msg3),123))

// получение log или результата
a.value // 123
a.written // Vector("msg1", "msg2", "msg3")

// получение log и результата
val (log, result) = a.run
```

Cats реализует Writer через WriterT
```scala
type Writer[W, A] = WriterT[Id, W, A]
```

Можно создавать определяя только один параметр log или результат
```scala
import cats.data.Writer  
import cats.syntax.applicative._ // for pure  

// пределяем только результат
type Logged[A] = Writer[Vector[String], A]  
123.pure[Logged] // WriterT((Vector(),123))

// определяем только log через Writer[Unit] используя синтакс tell
import cats.syntax.writer._  
Vector("msg1", "msg2", "msg3").tell // WriterT((Vector(msg1, msg2, msg3),()))
```

Применяя функцию flatMap мы добавляем лог в конец исходного лога, поэтому для хранения логов разумно использовать структуры данных эффективных для конкатенации и добавления в конец
```scala
import cats.data.Writer  
import cats.syntax.applicative._ // for pure  
import cats.syntax.writer._ // for writer  
  
type Logged[A] = Writer[Vector[String], A]  
  
val writer1 = for {  
	a <- 10.pure[Logged]  
	_ <- Vector("a", "b", "c").tell  
	b <- 32.writer(Vector("x", "y", "z"))  
} yield a + b // WriterT((Vector(a, b, c, x, y, z),42))

// преобразование log методом mapWritten
val writer2 = writer1.mapWritten(_.map(_.toUpperCase)) // WriterT((Vector(A, B, C, X, Y, Z),42))

// преобразование log и результата методом bimap (принимает 2 функциональных параметра)
val writer3 = writer1.bimap(  
	log => log.map(_.toUpperCase),  
	res => res * 100  
) // WriterT((Vector(A, B, C, X, Y, Z),4200))

// преобразование log и результата методом mapBoth (принимает 1 функцию с 2 параметрами)
val writer4 = writer1.mapBoth { (log, res) =>  
	val log2 = log.map(_ + "!")  
	val res2 = res * 1000  
	(log2, res2)  
} // WriterT((Vector(a!, b!, c!, x!, y!, z!),42000))

// обнуление log методом reset
val writer5 = writer1.reset // WriterT((Vector(),42))

// поменять местами log и результат методом swap
val writer6 = writer1.swap // WriterT((42,Vector(a, b, c, x, y, z)))
```

Пример
```scala
def slowly[A](body: => A) =  
	try body finally Thread.sleep(100)  
def factorial(n: Int): Int = {  
	val ans = slowly(if(n == 0) 1 else n * factorial(n - 1))  
	println(s"fact $n $ans")  
	ans  
}  
  
import scala.concurrent._  
import scala.concurrent.ExecutionContext.Implicits._  
import scala.concurrent.duration._  
  
Await.result(Future.sequence(Vector(  
	Future(factorial(5)),  
	Future(factorial(5))  
)), 5.seconds)
```

Перепишем через Writer чтобы в консоли было видно из какого потока мы пишем
```scala
def slowly[A](body: => A) =  
	try body finally Thread.sleep(100)  
  
import cats.data.Writer  
import cats.syntax.applicative._ // for pure  
import cats.syntax.writer._ // for tell  

// определили алиас для Writer чтобы использовать снтаксис pure
type Logged[A] = Writer[Vector[String], A]  
  
def factorial(n: Int): Logged[Int] =  
	for {  
		ans <- if (n == 0) {  
			1.pure[Logged]  
		} else {  
			slowly(factorial(n - 1).map(_ * n))  
		}  
		_ <- Vector(s"fact $n $ans").tell  
	} yield ans  
  
import scala.concurrent._  
import scala.concurrent.ExecutionContext.Implicits._  
import scala.concurrent.duration._  
  
Await.result(Future.sequence(Vector(  
	Future(factorial(5)),  
	Future(factorial(5))  
)).map(_.map(_.written)), 5.seconds)
```
## Монада Reader
Позволяет упорядочивать операции которые зависят от ввода. Reader оборачивает функции одного аргумента предоставляя полезные методы для их композиции (например, dependency injection). Если у нас есть ряд операций, зависящих от внешней конфигурации, то можно построить из них цепочку, используя Reader, чтобы создать одну операцию которая принимает конфигурацию как параметр и запускает программу в определенном порядке.

`Reader[A, B]` можно создать из функции A => B, используя конструктор Reader.apply

Пример
```scala
import cats.data.Reader  
  
final case class Cat(name: String, favoriteFood: String)  
  
val catName: Reader[Cat, String] = Reader(cat => cat.name)  
catName.run(Cat("Garfield", "lasagne")) // Garfield

// map расширяет вычисление пропуская его через функцию
val greetKitty: Reader[Cat, String] = catName.map(name => s"Hello ${name}")  
greetKitty.run(Cat("Heathcliff", "junk food")) // Hello Heathcliff

val feedKitty: Reader[Cat, String] =  
	Reader(cat => s"Have a nice bowl of ${cat.favoriteFood}")  

// flatMap позволяет комбинировать Reader зависящие от одного ввода
val greetAndFeed: Reader[Cat, String] =  
	for {  
		greet <- greetKitty  
		feed <- feedKitty  
	} yield s"$greet. $feed."  
  
greetAndFeed(Cat("Garfield", "lasagne")) // Hello Garfield. Have a nice bowl of lasagne.
```

Пример
```scala
import cats.data.Reader  
import cats.syntax.applicative._ // for pure  
  
final case class Db(usernames: Map[Int, String], passwords: Map[String, String])  
  
type DbReader[A] = Reader[Db, A]  
  
def findUsername(userId: Int): DbReader[Option[String]] =  
	Reader(db => db.usernames.get(userId))  
  
def checkPassword(username: String, password: String): DbReader[Boolean] =  
	Reader(db => db.passwords.get(username).contains(password))  
  
def checkLogin(userId: Int, password: String): DbReader[Boolean] = for {  
	username <- findUsername(userId)  
	passOk <- username.map { user =>  
		checkPassword(user, password)  
	}.getOrElse(false.pure[DbReader])  
} yield passOk  
  
val users = Map(  
	1 -> "dade",  
	2 -> "kate",  
	3 -> "margo"  
)  
  
val passwords = Map(  
	"dade" -> "zerocool",  
	"kate" -> "acidburn",  
	"margo" -> "secret"  
)  
  
val db = Db(users, passwords)  
  
checkLogin(1, "zerocool").run(db) // true
checkLogin(4, "davinci").run(db) // false
```
## Монада State
Позволяет передать дополнительное состояние как часть вычисления
Экземпляры State представляют атомарные операции состояния, которые можно связать друг с другом при помощи map и flatMap. Таким образом мы создаем изменяемое состояние в чисто функциональном стиле не используя действительного изменения

Экземпляры `State[S, A]` представляют собой функции S => (S, A), где S - тип состояния, А - тип результата. 

Эта функция делает 2 вещи:
- преобразует входное состояние в выходное
- вычисляет результат

Запустить монаду на выполнение можно методами:
- run (возвращает состояние и результат)
- runS (возвращает состояние)
- runA (возвращает результат)
Каждый из которых возвращает Eval для поддержки стеко-безопасности.
Метод value возвращает результат

```scala
import cats.data.State  
  
val a = State[Int, String]{ state =>  
	(state, s"The state is $state")  
}  
val (state, result) = a.run(10).value  
val justTheState = a.runS(10).value  
val justTheResult = a.runA(10).value
```

Методы map и flatMap передают состояние из одного экземпляра в другой, каждый экземпляр представляет собой атомное состояние трансформации и их комбинация представляет законченную последовательность изменений

```scala
import cats.data.State  
  
val step1 = State[Int, String] { num =>  
	val ans = num + 1  
	(ans, s"Result of step1: $ans")  
}  
val step2 = State[Int, String] { num =>  
	val ans = num * 2  
	(ans, s"Result of step2: $ans")  
}  
val both = for {  
	a <- step1  
	b <- step2  
} yield (a, b)  

val (state, result) = both.run(20).value
// 42
// (Result of step1: 21,Result of step2: 42)
```

Конструкторы
```scala
import cats.data.State  

// получает состояние как результат
val getDemo = State.get[Int]  
getDemo.run(10).value // (10,10)

// обновляет состояние и возвращает Unit как результат
val setDemo = State.set[Int](30)  
setDemo.run(10).value // (30,())

// игнорирует состояние и возвращает переданный результат
val pureDemo = State.pure[Int, String]("Result")  
pureDemo.run(10).value // (10,Result)

// получает состояние, преобразованное переданной фугънкцией
val inspectDemo = State.inspect[Int, String](x => s"${x}!")  
inspectDemo.run(10).value // (10,10!)

// обновляет состояние, используя переданную функцию
val modifyDemo = State.modify[Int](_ + 1)  
modifyDemo.run(10).value // (11,())

import State._  
  
val program: State[Int, (Int, Int, Int)] = for {  
	a <- get[Int]  
	_ <- set[Int](a + 1)  
	b <- get[Int]  
	_ <- modify[Int](_ + 1)  
	c <- inspect[Int, Int](_ * 1000)  
} yield (a, b, c)  
  
val (state, result) = program.run(1).value
// 3
// (1,2,3000)
```

Калькулятор постфиксный
```scala
import cats.data.State  
type CalcState[A] = State[List[Int], A]  
def evalOne(sym: String): CalcState[Int] =  
	sym match {  
		case "+" => operator(_ + _)  
		case "-" => operator(_ - _)  
		case "*" => operator(_ * _)  
		case "/" => operator(_ / _)  
		case num => operand(num.toInt)  
	}  
  
def operand(num: Int): CalcState[Int] =  
	State[List[Int], Int] { stack =>  
		(num :: stack, num)  
	}  
  
def operator(func: (Int, Int) => Int): CalcState[Int] =  
	State[List[Int], Int] {  
		case b :: a :: tail =>  
			val ans = func(a, b)  
			(ans :: tail, ans)  
		case _ =>  
			sys.error("Fail!")  
	}  
  
evalOne("42").runA(Nil).value // 42

val program = for {  
	_ <- evalOne("1")  
	_ <- evalOne("2")  
	ans <- evalOne("+")  
} yield ans  
  
program.runA(Nil).value // 3

import cats.syntax.applicative._ // for pure  
  
def evalAll(input: List[String]): CalcState[Int] = 
	input.foldLeft(0.pure[CalcState]){ (a, b) =>  
		a.flatMap(_ => evalOne(b))  
	}  
  
evalAll(List("1", "2", "+", "3", "*")).runA(Nil).value // 9

val biggerProgram = for {  
	_ <- evalAll(List("1", "2", "+"))  
	_ <- evalAll(List("3", "4", "+"))  
	ans <- evalOne("*")  
} yield ans  
  
biggerProgram.runA(Nil).value // 21

def evalInput(input: String): Int = evalAll(input.split(" ").toList).runA(Nil).value  
  
evalInput("1 2 + 3 4 + *") // 21
```

## Пользовательские монады

Чтобы создать монаду необходимо реализовать 3 функции: flatMap, pure, и tailRecM 

```scala
import cats.Monad
import scala.annotation.tailrec

val optionMonad = new Monad[Option] {
  def flatMap[A, B](opt: Option[A])(fn: A => Option[B]): Option[B] =
    opt flatMap fn

  def pure[A](opt: A): Option[A] = Some(opt)

  @tailrec  def tailRecM[A, B](a: A)
      (fn: A => Option[Either[A, B]]): Option[B] =
    fn(a) match {
      case None           => None
      case Some(Left(a1)) => tailRecM(a1)(fn)
      case Some(Right(b)) => Some(b)
	} 
}
```
Метод tailRecM является оптимизацией чтобы ограничить пространство в стеке занимаемое внутренними вызовами flatMap, метод рекурсивно вызывает себя пока fn не вернет Right.

```scala
import cats.Monad  
import cats.syntax.flatMap._ // For flatMap  
  
def retry[F[_] : Monad, A](start: A)(f: A => F[A]): F[A] =  
	f(start).flatMap { a =>  
		retry(a)(f)  
	}  
  
retry[Option, Int](100)(a => if (a == 0) None else Some(a - 1)) // None
retry[Option, Int](100000)(a => if (a == 0) None else Some(a - 1)) // StackOverflowError
```

Перепишем используя tailRecM
```scala
import cats.Monad
import cats.syntax.functor._ // for map  
  
def retryTailRecM[F[_] : Monad, A](start: A)(f: A => F[A]): F[A] = Monad[F].tailRecM(start) { a =>  
	f(a).map(a2 => Left(a2))  
}  
  
retryTailRecM[Option, Int](100000)(a => if (a == 0) None else Some(a - 1)) // None
```

Перепишем используя iterateWhileM
```scala
import cats.Monad  
import cats.syntax.monad._ // for iterateWhileM  
  
def retryM[F[_] : Monad, A](start: A)(f: A => F[A]): F[A] =  
	start.iterateWhileM(f)(_ => true)  
  
retryM[Option, Int](100000)(a => if (a == 0) None else Some(a - 1))  
  
retryM[Option, Int](100000)(a => if (a == 0) None else Some(a - 1)) // None
```

Монада Tree
```scala
sealed trait Tree[+A]  
  
final case class Branch[A](left: Tree[A], right: Tree[A]) extends Tree[A]  
final case class Leaf[A](value: A) extends Tree[A]  
  
def branch[A](left: Tree[A], right: Tree[A]): Tree[A] = Branch(left, right)  
def leaf[A](value: A): Tree[A] = Leaf(value)  
  
import cats.Monad  
  
implicit val treeMonad = new Monad[Tree] {  
	def pure[A](value: A): Tree[A] =  
		Leaf(value)  
	  
	def flatMap[A, B](tree: Tree[A])  
	(func: A => Tree[B]): Tree[B] =  
		tree match {  
			case Branch(l, r) =>  
				Branch(flatMap(l)(func), flatMap(r)(func))  
			case Leaf(value) =>  
				func(value)  
		}  
	  
	def tailRecM[A, B](a: A)(func: A => Tree[Either[A, B]]): Tree[B] = 
		flatMap(func(a)) {  
			case Left(value) =>  
				tailRecM(value)(func)  
			case Right(value) =>  
				Leaf(value)  
		}  
}  
  
import cats.syntax.flatMap._ // for flatMap  
  
branch(leaf(100), leaf(200)).  
flatMap(x => branch(leaf(x - 1), leaf(x + 1))) // Branch(Branch(Leaf(99),Leaf(101)),Branch(Leaf(199),Leaf(201)))
```

Версия tailrec
```scala
import cats.Monad
import scala.annotation.tailrec  
  
implicit val treeMonad = new Monad[Tree] {  
	def pure[A](value: A): Tree[A] =  
		Leaf(value)  
  
	def flatMap[A, B](tree: Tree[A])  
	(func: A => Tree[B]): Tree[B] =  
		tree match {  
			case Branch(l, r) => 
				Branch(flatMap(l)(func), flatMap(r)(func))  
			case Leaf(value) =>  
				func(value)  
		}  
  
def tailRecM[A, B](arg: A)  
(func: A => Tree[Either[A, B]]): Tree[B] = {  
	@tailrec  
	def loop(  
	open: List[Tree[Either[A, B]]],  
	closed: List[Option[Tree[B]]]): List[Tree[B]] = open match {  
		case Branch(l, r) :: next =>  
			loop(l :: r :: next, None :: closed)  
		case Leaf(Left(value)) :: next =>  
			loop(func(value) :: next, closed)  
		case Leaf(Right(value)) :: next =>  
			loop(next, Some(pure(value)) :: closed)  
		case Nil =>  
			closed.foldLeft(Nil: List[Tree[B]]) { (acc, maybeTree) =>  
				maybeTree.map(_ :: acc).getOrElse {  
					val left :: right :: tail = acc  
					branch(left, right) :: tail  
				}  
			}  
	}  
	loop(List(func(arg)), Nil).head  
	}  
}

import cats.syntax.functor._ // for map  
import cats.syntax.flatMap._ // for flatMap  
  
for {  
	a <- branch(leaf(100), leaf(200))  
	b <- branch(leaf(a - 10), leaf(a + 10))  
	c <- branch(leaf(b - 1), leaf(b + 1))  
} yield c
// Branch(Branch(Branch(Leaf(89),Leaf(91)),Branch(Leaf(109),Leaf(111))),Branch(Branch(Leaf(189),Leaf(191)),Branch(Leaf(209),Leaf(211))))
```
## Монады трансформеры
Служат для композиции нескольких монад
Трансформер представляет из себя внутреннюю монаду в стеке, а его первый параметр - внешнюю монаду, второй параметр - тип, который используем в монадах. Например `OptionT[List, A]` это `List[Option[A]]` то есть мы строим стек монад изнутри наружу

```scala
import cats.data.OptionT  
import cats.syntax.applicative._ // for pure  
  
type ListOption[A] = OptionT[List, A]  
  
val result1: ListOption[Int] = OptionT(List(Option(10)))  
val result2: ListOption[Int] = 32.pure[ListOption]  
  
result1.flatMap { (x: Int) =>  
	result2.map { (y: Int) =>  
		x + y  
	}  
} // OptionT(List(Some(42)))
```

Если нам надо обернуть Option в Either, то нужно использовать OptionT, но Either имеет 2 тайп параметра, а монада - только один, поэтому нам нужен тайп алиас чтобы преобразовать тайп конструктор в нужную форму
```scala
type ErrorOr[A] = Either[String, A]
type ErrorOrOption[A] = OptionT[ErrorOr, A]

import cats.data.OptionT  
import cats.syntax.applicative._ // for pure  
  
val a = 10.pure[ErrorOrOption]  
val b = 32.pure[ErrorOrOption]  
  
val c = a.flatMap(x => b.map(y => x + y)) // OptionT(Right(Some(42)))
c.value // Right(Some(42))
c.value.map(_.getOrElse(-1)) // Right(42)
```

Если у нас Future от Either от Option, то нам нужно создать OptionT от EitherT от Future
```scala
import cats.data.{EitherT, OptionT}  
import cats.syntax.applicative._  
  
import scala.concurrent.ExecutionContext.Implicits.global  
import scala.concurrent.Future  
  
type FutureEither[A] = EitherT[Future, String, A]  
type FutureEitherOption[A] = OptionT[FutureEither, A]  
  
val futureEitherOr: FutureEitherOption[Int] =  
	for {  
		a <- 10.pure[FutureEitherOption]  
		b <- 32.pure[FutureEitherOption]  
	} yield a + b // OptionT(EitherT(Future(...)))

futureEitherOr.value // EitherT(Future(...))
futureEitherOr.value.value // Future(...)
Await.result(futureEitherOr.value.value, 1.seconds) // Right(Some(42))
```

Монады определенные через трансформеры
```scala
type Reader[E, A] = ReaderT[Id, E, A] // = Kleisli[Id, E, A]
type Writer[W, A] = WriterT[Id, W, A]
type State[S, A]  = StateT[Id, S, A]
```

Пример
```scala
import cats.data.EitherT  
  
import scala.concurrent.ExecutionContext.Implicits.global  
import scala.concurrent.duration._  
import scala.concurrent.{Await, Future}  
  
type Response[A] = EitherT[Future, String, A]  
  
def getPowerLevel(ally: String): Response[Int] = {  
	powerLevels.get(ally) match {  
		case Some(avg) => EitherT.right(Future(avg))  
		case None => EitherT.left(Future(s"$ally unreachable"))  
	}  
}  
  
def canSpecialMove(ally1: String, ally2: String): Response[Boolean] = for {  
	power1 <- getPowerLevel(ally1)  
	power2 <- getPowerLevel(ally2)  
} yield (power1 + power2) > 15  
  
def tacticalReport(ally1: String, ally2: String): String = {  
	val stack = canSpecialMove(ally1, ally2).value  
	  
	Await.result(stack, 1.second) match {  
		case Left(msg) =>  
			s"Comms error: $msg"  
		case Right(true) =>  
			s"$ally1 and $ally2 are ready to roll out!"
		case Right(false) =>  
			s"$ally1 and $ally2 need a recharge."
	}  
}  
  
val powerLevels = Map(  
	"Jazz" -> 6,  
	"Bumblebee" -> 8,  
	"Hot Rod" -> 10  
)  
  
tacticalReport("Jazz", "Bumblebee") // Jazz and Bumblebee need a recharge.
tacticalReport("Bumblebee", "Hot Rod") // Bumblebee and Hot Rod are ready to roll out!
tacticalReport("Jazz", "Ironhide") // Comms error: Ironhide unreachable
```
# Semigroupal
 Позволяет объединять контексты. 
 Если у нас есть `F[A]` и `F[B]` (независимые друг от друга), то `Semigroupal[F]` позволяет объединить их в `F[(A, B)]`
```scala
trait Semigroupal[F[_]] {
  def product[A, B](fa: F[A], fb: F[B]): F[(A, B)]
}
```

Semigroup позволяет объединять значения, а Semigroupal - контексты.
```scala
import cats.Semigroupal  
  
Semigroupal[Option].product(Some(123), Some("abc")) // Some((123,abc))
Semigroupal[Option].product(None, Some("abc")) // None

// обобщение product на раличные арности (от 2 до 22)
Semigroupal.tuple3(Option(1), Option(2), Option(3)) // Some((1,2,3))
Semigroupal.tuple3(Option(1), Option(2), Option.empty[Int]) // None

// применить функцию к аргументам (арности от 2 до 22)
Semigroupal.map3(Option(1), Option(2), Option(3))(_ + _ + _) // Some(6)
Semigroupal.map2(Option(1), Option.empty[Int])(_ + _) // None
```
## Закон
Для Semigroupal должен выполняться закон:
- product должен быть ассоциативным
	- product(a, product(b, c)) == product(product(a, b), c)

## Интерфейс Syntax
```scala
import cats.syntax.apply._  
  
(Option(123), Option("abc")).tupled // Some((123,abc))
(Option(123), Option("abc"), Option(true)).tupled // Some((123,abc,true))

final case class Cat(name: String, born: Int, color: String)  

(Option("Garfield"), Option(1978), Option("Orange & black")).mapN(Cat.apply) // Some(Cat(Garfield,1978,Orange & black))
```

mapN - принимает implicit Functor и function нужной арности чтобы объединить переданные значения. Внутри использует Semigroupal чтобы извлечь значения из Option и Functor чтобы передать значения в функцию.

mapN имеет проверку типов и их количество
```scala
val add: (Int, Int) => Int = (a, b) => a + b

(Option(1), Option(2), Option(3)).mapN(add) // compile error
(Option("cats"), Option(true)).mapN(add) // compile error
```

Соединим моноиды при помощи Invariant
```scala
import cats.Monoid  
import cats.syntax.apply._ // for imapN  
import cats.instances.invariant._ // for Semigroupal  
  
final case class Cat(name: String, yearOfBirth: Int, favoriteFoods: List[String])  
  
val tupleToCat: (String, Int, List[String]) => Cat = Cat.apply  
val catToTuple: Cat => (String, Int, List[String]) = cat => (cat.name, cat.yearOfBirth, cat.favoriteFoods)  
implicit val catMonoid: Monoid[Cat] = (  
	Monoid[String],  
	Monoid[Int],  
	Monoid[List[String]]  
).imapN(tupleToCat)(catToTuple)

// теперь кошек можно сложить ))

import cats.syntax.semigroup._ // for |+|   
  
val garfield = Cat("Garfield", 1978, List("Lasagne"))  
val heathcliff = Cat("Heathcliff", 1988, List("Junk Food"))  
  
println(garfield |+| heathcliff) // Cat(GarfieldHeathcliff,3966,List(Lasagne, Junk Food))
```

Semigroupal для Future
```scala
import cats.Semigroupal 
import cats.syntax.apply._  
  
import scala.concurrent.ExecutionContext.Implicits.global  
import scala.concurrent.{Await, Future}  
import scala.concurrent.duration._   
  
val futurePair = Semigroupal[Future].product(Future("Hello"), Future(123))  
Await.result(futurePair, 1.second) // (Hello,123)

case class Cat(name: String, yearOfBirth: Int, favoriteFoods: List[String])  
  
val futureCat = (  
	Future("Garfield"),  
	Future(1978),  
	Future(List("Lasagne"))  
).mapN(Cat.apply)  
  
Await.result(futureCat, 1.second) // Cat(Garfield,1978,List(Lasagne))
```

Semigroupal для List создает декартово произведение
```scala
import cats.Semigroupal  
import cats.syntax.apply._ // for tupled
  
Semigroupal[List].product(List(1, 2), List(3, 4)) // List((1,3), (1,4), (2,3), (2,4))
(List(1, 2), List(3, 4)).tupled // List((1,3), (1,4), (2,3), (2,4))
```

Мы получаем такой результат потому что List - это монада и тогда
```scala
import cats.Monad
import cats.syntax.functor._ // for map
import cats.syntax.flatMap._ // for flatMap

def product[F[_]: Monad, A, B](fa: F[A], fb: F[B]): F[(A, B)] =
  fa.flatMap(a =>  
		fb.map(b =>  
			(a, b)
		) 
	)

// or
def product[F[_]: Monad, A, B](x: F[A], y: F[B]): F[(A, B)] =
  for {
	a <- x
    b <- y
  } yield (a, b)
```

Semigroupal для Either
```scala
import cats.Semigroupal  
  
type ErrorOr[A] = Either[Vector[String], A]  
  
Semigroupal[ErrorOr].product(  
	Left(Vector("Error 1")),  
	Left(Vector("Error 2"))  
) // Left(Vector(Error 1))
```
# Parallel
Позволяет тип, имеющий экземпляр монады, преобразовать в экземпляр аппликатива или Semigroupal. Это бывает необходимо когда нужно получить альтернативное поведение от того, что дает например Semigroupal, applicative для Either позволяет собрать все ошибки выполнения.

Позволяет получить отличное от Semigroupal поведение. 
Например чтобы собрать все ошибки при соединении двух Either
```scala
import cats.syntax.parallel._ // for parTupled  
  
type ErrorOr[A] = Either[Vector[String], A]  
val error1: ErrorOr[Int] = Left(Vector("Error 1"))  
val error2: ErrorOr[Int] = Left(Vector("Error 2"))  
  
(error1, error2).parTupled // Left(Vector(Error 1, Error 2))

val success1: ErrorOr[Int] = Right(1)  
val success2: ErrorOr[Int] = Right(2)  
  
val addTwo = (x: Int, y: Int) => x + y

(success1, success2).parMapN(addTwo) // Right(3)
```

Версия для List сжимает 2 списка
```scala
import cats.syntax.parallel._  
  
(List(1, 2), List(3, 4)).parTupled // List((1,3), (2,4))
```

Устройство Parallel
```scala
import cats.{Applicative, Monad, ~>}  
  
trait Parallel[M[_]] {  
	type F[_]  
	def applicative: Applicative[F]  
	def monad: Monad[M]  
	def parallel: ~>[M, F]  
}
```
Если у нас есть экземпляр Parallel для тайп-конструктора М, то для М должен быть 
- экземпляр монады
- тайп-конструктор F, имеющий экземпляр Applicative и преобразование из М в F

`~>` - это тайп-алиас для FunctionK и он выполняет преобразование из М в F

Функция FunctionK M ~> F функция преобразующая значение `M[A]` в значение `F[A]`

```scala
import cats.arrow.FunctionK  
  
object optionToList extends FunctionK[Option, List] {  
	def apply[A](fa: Option[A]): List[A] =  
		fa match {  
			case None => List.empty[A]  
			case Some(a) => List(a)  
		}  
}  
  
optionToList(Some(1)) // List(1)
optionToList(None) // List()
```

# Applicative  
(или Applicative functor)
Расширяет Semigroupal и Functor. Позволяет применять функции к параметрам внутри контекста

Модели аппликативов используют 2 тайп класса Apply и Applicative
```scala
trait Apply[F[_]] extends Semigroupal[F] with Functor[F] { 
	// применяет параметр fa к функции ff внутри контекста F
	def ap[A, B](ff: F[A => B])(fa: F[A]): F[B]
	// определен через ap и map. Каждый из product, ap и map могут определены через 2 других
	def product[A, B](fa: F[A], fb: F[B]): F[(A, B)] =
	    ap(map(fa)(a => (b: B) => (a, b)))(fb)
}
// Applicative относится к Apply как Monoid к Semigroup
trait Applicative[F[_]] extends Apply[F] {
	def pure[A](a: A): F[A]
}
```

Иерархия тайп классов
![[TypeClassesHierarhy.png|center|200]]

Apply определяет product через ap и map
Monad определяет product, ap и map через pure и flatMap

- recap - резюме, итоги
- fold - складывать
- handful - небольшое количество
- treat - рассматривать
- facsimile - копия
- leverage - использовать
- poll - опрашивать
- uptime - время работы
- unwieldy - громоздкий
- tailor‐made - специальный
- squint - прищуриться
# Foldable
Предоставляет foldLeft и foldRight операции над типами последовательностей

```scala
def show[A](list: List[A]): String =  
list.foldLeft("nil")((accum, item) => s"$item then $accum")  
  
show(Nil) // nil
show(List(1, 2, 3)) // 3 then 2 then 1 then nil
```

foldLeft и foldRight эквивалентны для ассоциативных операций
```scala
List(1, 2, 3).foldLeft(0)(_ + _) // 6
List(1, 2, 3).foldRight(0)(_ + _) // 6

List(1, 2, 3).foldLeft(0)(_ - _) // -6
List(1, 2, 3).foldRight(0)(_ - _) // 2

List(1, 2, 3).foldLeft(List.empty[Int])((a, i) => i :: a) // List(3, 2, 1)
List(1, 2, 3).foldRight(List.empty[Int])((i, a) => i :: a) // List(1, 2, 3)

def map[A, B](list: List[A])(func: A => B): List[B] =  
	list.foldRight(List.empty[B]) { (item, accum) =>  
		func(item) :: accum  
}  
map(List(1, 2, 3))(_ * 2) // List(2, 4, 6)

def flatMap[A, B](list: List[A])(func: A => List[B]): List[B] =  
	list.foldRight(List.empty[B]) { (item, acc) =>  
		func(item) ::: acc  
}  
  
flatMap(List(1, 2, 3))(a => List(a, a * 10, a * 100)) // List(1, 10, 100, 2, 20, 200, 3, 30, 300)

def filter[A](list: List[A])(func: A => Boolean): List[A] =  
	list.foldRight(List.empty[A]) { (item, accum) =>  
		if (func(item)) item :: accum else accum  
}  

filter(List(1, 2, 3))(_ % 2 == 1) // List(1, 3)

def sumWithNumeric[A](list: List[A])  
(implicit numeric: Numeric[A]): A =  
	list.foldRight(numeric.zero)(numeric.plus)  
	
sumWithNumeric(List(1, 2, 3)) // 6

import cats.Monoid  
def sumWithMonoid[A](list: List[A])  
(implicit monoid: Monoid[A]): A =  
	list.foldRight(monoid.empty)(monoid.combine)  
  
sumWithMonoid(List(1, 2, 3)) // 6
```

Foldable имеет готовые реализации для List, Vector, LazyList, and Option
```scala
import cats.Foldable  
  
Foldable[List].foldLeft(List(1, 2, 3), 0)(_ + _) // 6
Foldable[Option].foldLeft(Option(123), 10)(_ * _) // 1230
```

foldRight реализован через монаду Eval для стекобезопасности
```scala
def foldRight[A, B](fa: F[A], lb: Eval[B])
	               (f: (A, Eval[B]) => Eval[B]): Eval[B]
```
Пример
```scala
import cats.{Eval, Foldable}  
  
def bigData = (1 to 100000).to(LazyList)  
  
val eval: Eval[Long] = Foldable[LazyList].  
	foldRight(bigData, Eval.now(0L)) { (num, eval) =>  
		eval.map(_ + num)  
	}  
eval.value // 5000050000
```

Foldable предоставляет множество полезных методов: find, exists, forall, toList, isEmpty, nonEmpty и так далее
```scala
Foldable[Option].nonEmpty(Option(42)) // true
Foldable[List].find(List(1, 2, 3))(_ % 2 == 0) // Some(2)
```

а также 
- combineAll (и его алиас fold) соединяет все элементы в последовательности используя моноид
- foldMap применяет функцию на последовательность и соединяет все элементы в последовательности используя моноид
- compose позволяет складывать вложенные последовательности
```scala
Foldable[List].combineAll(List(1, 2, 3)) // 6
Foldable[List].fold(List(1, 2, 3)) // 6
Foldable[List].foldMap(List(1, 2, 3))(_.toString) // 123

val ints = List(Vector(1, 2, 3), Vector(4, 5, 6))
(Foldable[List] compose Foldable[Vector]).combineAll(ints) // 21
```

## Интерфейс Syntax
```scala
import cats.syntax.foldable._ // for combineAll and foldMap  
  
List(1, 2, 3).combineAll // 6
List(1, 2, 3).foldMap(_.toString) // 123
```

Scala использует Foldable версию, только если метод явно не доступен на контейнере
```scala
List(1, 2, 3).foldLeft(0)(_ + _) // foldLeft из List

def sum[F[_]: Foldable](values: F[Int]): Int =
	values.foldLeft(0)(_ + _) // foldLeft из Foldable
```

# Traverse
Пример стандартного подхода
```scala
import scala.concurrent._  
import scala.concurrent.duration._  
import scala.concurrent.ExecutionContext.Implicits.global  
  
val hostnames = List(  
	"alpha.example.com",  
	"beta.example.com",  
	"gamma.demo.com"  
)  
  
def getUptime(hostname: String): Future[Int] =  
	Future(hostname.length * 60) 
  
val allUptimes: Future[List[Int]] = hostnames.foldLeft(Future(List.empty[Int])) {  
	(accum, host) =>  
		val uptime = getUptime(host)  
		for {  
			accum <- accum  
			uptime <- uptime  
		} yield accum :+ uptime  
}  
  
Await.result(allUptimes, 1.second) // List(1020, 960, 840)
```

Используя Future.traverse (принцип его работы описан в примере выше)
- начинаем с `List[A]`
- даем функцию `A => Future[B]`
- получаем результат `Future[List[B]]`
```scala
val allUptimes: Future[List[Int]] = Future.traverse(hostnames)(getUptime)

Await.result(allUptimes, 1.second) // List(1020, 960, 840)
```

Используя Future.sequence
- начинаем с `List[Future[A]]`
- получаем `Future[List[A]]`
```scala
object Future {  
	def sequence[B](futures: List[Future[B]]): Future[List[B]] =
	    traverse(futures)(identity)
//...
}
```
Перепишем traverse с помощью Applicative
```scala
import cats.Applicative  
import cats.syntax.applicative._ // for pure  
import cats.syntax.apply._ // for mapN

// Future(List.empty[Int]) == List.empty[Int].pure[Future]
def listTraverse[F[_] : Applicative, A, B]  
(list: List[A])(func: A => F[B]): F[List[B]] =  
	list.foldLeft(List.empty[B].pure[F]) {  
		(accum, item) => (accum, func(item)).mapN(_ :+ _)  
	}  

val totalUptime = listTraverse(hostnames)(getUptime)

Await.result(totalUptime, 1.second) // List(1020, 960, 840)
  
def listSequence[F[_] : Applicative, B]  
(list: List[F[B]]): F[List[B]] =  
	listTraverse(list)(identity)

listSequence(List(Vector(1, 2), Vector(3, 4))) // Vector(List(1, 3), List(1, 4), List(2, 3), List(2, 4))
listSequence(List(Vector(1, 2), Vector(3, 4), Vector(5, 6))) // Vector(List(1, 3, 5), List(1, 3, 6), List(1, 4, 5), List(1, 4, 6), List(2, 3, 5), List(2, 3, 6), List(2, 4, 5), List(2, 4, 6))

def process(inputs: List[Int]) =  
	listTraverse(inputs)(n => if (n % 2 == 0) Some(n) else None)  
  
process(List(2, 4, 6)) // Some(List(2, 4, 6)) 
process(List(1, 2, 3)) // None

import cats.data.Validated  
  
type ErrorsOr[A] = Validated[List[String], A]

def process(inputs: List[Int]): ErrorsOr[List[Int]] = listTraverse(inputs) { n =>  
	if (n % 2 == 0) {  
		Validated.valid(n)  
	} else {  
		Validated.invalid(List(s"$n is not even"))  
	}  
}  
  
process(List(2, 4, 6)) // Valid(List(2, 4, 6))
process(List(1, 2, 3)) // Invalid(List(1 is not even, 3 is not even))
```

Перепишем listTraverse для любого типа контейнера
```scala
trait Traverse[F[_]] {  
	def traverse[G[_] : Applicative, A, B](inputs: F[A])(func: A => G[B]): G[F[B]]  
	  
	def sequence[G[_] : Applicative, B]  
	(inputs: F[G[B]]): G[F[B]] = traverse(inputs)(identity)  
}
```

Cats предоставляет Traverse для разных контейнеров: List, Vector, Stream, Option, Either и тд
```scala
val totalUptime: Future[List[Int]] = Traverse[List].traverse(hostnames)(getUptime)  
Await.result(totalUptime, 1.second) // List(1020, 960, 840)

val numbers = List(Future(1), Future(2), Future(3))  
val numbers2: Future[List[Int]] = Traverse[List].sequence(numbers)  
Await.result(numbers2, 1.second) // List(1, 2, 3)
```

## Интерфейс Syntax
```scala
import cats.syntax.traverse._ // for sequence and traverse  
Await.result(hostnames.traverse(getUptime), 1.second) // List(1020, 960, 840)
Await.result(numbers.sequence, 1.second) // List(1, 2, 3)
```
# Asynchronous Code

Абстрагироование от типа контейнера для тестирования
```scala
import cats.{Applicative, Id}  
import cats.syntax.functor._ // for map  
import cats.syntax.traverse._  
  
import scala.concurrent.Future
  
trait UptimeClient[F[_]] {  
	def getUptime(hostname: String): F[Int]  
}  
  
class RealUptimeClient(hosts: Map[String, Int]) extends UptimeClient[Future] {  
	def getUptime(hostname: String): Future[Int] =  
		Future.successful(hosts.getOrElse(hostname, 0))  
}  
  
class TestUptimeClient(hosts: Map[String, Int]) extends UptimeClient[Id] {  
	def getUptime(hostname: String): Int =  
		hosts.getOrElse(hostname, 0)  
}  

// or UptimeService[F[_]: Applicative](client: UptimeClient[F])
class UptimeService[F[_]](client: UptimeClient[F])(implicit F: Applicative[F]) {  
	def getTotalUptime(hostnames: List[String]): F[Int] =  
		hostnames.traverse(client.getUptime).map(_.sum)  
} 
  
def testTotalUptime() = {  
	val hosts = Map("host1" -> 10, "host2" -> 6)  
	val client = new TestUptimeClient(hosts)  
	val service = new TestUptimeService(client)  
	val actual = service.getTotalUptime(hosts.keys.toList)  
	val expected = hosts.values.sum  
	assert(actual == expected)  
}
```

# Map‐Reduce
Это программная модель которая производит преобразования данных методом map и их сложение (reduce) методом fold. Метод map не накладывает никаких ограничений на порядок элементов, а чтобы fold обладал свойством параллельности от него надо потребовать ассоциативность
```
reduce(a1, reduce(a2, a3)) == reduce(reduce(a1, a2), a3)
```
и fold должен также иметь начальный элемент, который не влияет на результат вычисления, то есть обладает свойством identity:
```
reduce(seed, a1) == reduce(a1, seed) == a1
```
Такими свойствами обладает Monoid

```scala
import cats.Monoid  
import cats.syntax.semigroup._ // for |+|

def foldMap[A, B: Monoid](values: Vector[A])(func: A => B): B = values.map(func).foldLeft(Monoid[B].empty)(_ |+| _)  

// или
def foldMap[A, B: Monoid](values: Vector[A])(func: A => B): B = values.foldLeft(Monoid[B].empty)(_ |+| func(_))

foldMap(Vector(1, 2, 3))(identity) // 6
foldMap(Vector(1, 2, 3))(_.toString + "! ") // 1! 2! 3! 
foldMap("Hello world!".toVector)(_.toString.toUpperCase) // HELLO WORLD!
```

- partition - разделить
- negligible - незначительный
- stochastic - вероятностный, случайный
- impasse - тупик
	- We are at an impasse - мы в тупике

Future запускается на thread pool, определенном параметром ExecutionContext.
ExecutionContext.Implicits.global определяет пул потоков, по одному потоку на ядро.
```scala
val f1 = Future {  
(1 to 100).toList.foldLeft(0)(_ + _)  
}  // Future[Int]
  
import cats.Monad  
val f2 = Monad[Future].pure(42) // Future[Int]
  
import cats.Monoid  
val f3 = Monoid[Future[Int]].combine(Future(1), Future(2)) // Future[Int]

val f4 = Future.sequence(List(Future(1), Future(2), Future(3))) // Future[List[Int]]
  
import cats.syntax.traverse._  
val f5 = List(Future(1), Future(2), Future(3)).sequence // Future[List[Int]]
```

Узнать количество ядер на хосте
```
Runtime.getRuntime.availableProcessors
```

```scala
import cats.Monoid  
import cats.syntax.semigroup._  
  
import scala.concurrent.ExecutionContext.Implicits.global  
import scala.concurrent.duration._  
import scala.concurrent.{Await, Future}  
  
def foldMap[A, B: Monoid](values: Vector[A])(func: A => B): B = values.foldLeft(Monoid[B].empty)(_ |+| func(_))  
  
def parallelFoldMap[A, B : Monoid](values: Vector[A])(func: A => B): Future[B] = {  
	// Calculate the number of items to pass to each CPU:  
	val numCores = Runtime.getRuntime.availableProcessors  
	// ceil округляет в большую сторону
	val groupSize = (1.0 * values.size / numCores).ceil.toInt  
	  
	// Create one group for each CPU:  
	val groups: Iterator[Vector[A]] = values.grouped(groupSize)  
	  
	// Create a future to foldMap each group:  
	val futures: Iterator[Future[B]] = groups.map(gr => Future(foldMap(gr)(func)))  
	  
	// foldMap over the groups to calculate a final result:  
	Future.sequence(futures) map { iterable =>  
		iterable.foldLeft(Monoid[B].empty)(_ |+| _)  
	}  
}  
  
val result: Future[Int] = parallelFoldMap((1 to 1000000).toVector)(identity)  
Await.result(result, 1.second) // 1784293664
```

Используя стандартный foldMap из Foldable
```scala
import cats.Monoid  
  
import scala.concurrent.ExecutionContext.Implicits.global  
import scala.concurrent.duration._  
import scala.concurrent.{Await, Future}  
  
def parallelFoldMap[A, B: Monoid](values: Vector[A])(func: A => B): Future[B] = {  
	// Calculate the number of items to pass to each CPU:  
	val numCores = Runtime.getRuntime.availableProcessors  
	// ceil округляет в большую сторону  
	val groupSize = (1.0 * values.size / numCores).ceil.toInt  
	  
	import cats.syntax.foldable._ // for foldMap and combineAll
	import cats.syntax.traverse._ // for traverse  
	values  
		.grouped(groupSize)  
		.toVector  
		.traverse(group => Future(group.foldMap(func)))  
		.map(_.combineAll)  
}  
  
val result: Future[Int] = parallelFoldMap((1 to 1000).toVector)(_ * 1000)  
println(Await.result(result, 1.second)) // 500500000
```
# Data Validation
```scala
import cats.Semigroup  
import cats.syntax.either._ // for asLeft and asRight  
import cats.syntax.semigroup._ // for |+|  
  
final case class CheckF[E, A](func: A => Either[E, A]) {  
	def apply(a: A): Either[E, A] = func(a)  
  
	def and(that: CheckF[E, A])(implicit s: Semigroup[E]): CheckF[E, A] =  
		CheckF { a =>  
			(this(a), that(a)) match {  
				case (Left(e1), Left(e2)) => (e1 |+| e2).asLeft  
				case (Left(e), Right(_)) => e.asLeft  
				case (Right(_), Left(e)) => e.asLeft  
				case (Right(_), Right(_)) => a.asRight  
			}  
		} 
}  
  
val a: CheckF[List[String], Int] = CheckF { v =>  
	if (v > 2) v.asRight  
	else List("Must be > 2").asLeft  
}  
  
val b: CheckF[List[String], Int] = CheckF { v =>  
	if (v < -2) v.asRight  
	else List("Must be < -2").asLeft  
}  
val check: CheckF[List[String], Int] = a and b  
  
check(5) // Left(List(Must be < -2))
check(0) // Left(List(Must be > 2, Must be < -2))
```

Решение через ADT
```scala
import cats.Semigroup  
import cats.syntax.either._ // for asLeft and asRight  
import cats.syntax.semigroup._ // for |+|  
  
sealed trait Check[E, A] {  
	import Check._  
	  
	def and(that: Check[E, A]): Check[E, A] = And(this, that)  
	  
	def apply(a: A)(implicit s: Semigroup[E]): Either[E, A] = this match {  
		case Pure(func) => func(a)  
		case And(left, right) =>  
			(left(a), right(a)) match {  
				case (Left(e1), Left(e2)) => (e1 |+| e2).asLeft  
				case (Left(e), Right(_)) => e.asLeft  
				case (Right(_), Left(e)) => e.asLeft  
				case (Right(_), Right(_)) => a.asRight  
			}  
	}  
}  
  
object Check {  
final case class And[E, A](left: Check[E, A], right: Check[E, A]) extends Check[E, A]  
  
final case class Pure[E, A](func: A => Either[E, A]) extends Check[E, A]  
  
def pure[E, A](f: A => Either[E, A]): Check[E, A] = Pure(f)  
}  
  
val c: Check[List[String], Int] = Check.pure { v =>  
	if (v > 2) v.asRight  
	else List("Must be > 2").asLeft  
}  
  
val d: Check[List[String], Int] = Check.pure { v =>  
	if (v < -2) v.asRight  
	else List("Must be < -2").asLeft  
}  
val check2: Check[List[String], Int] = c and d  
  
check2(5) // Left(List(Must be < -2))
check2(0) // Left(List(Must be > 2, Must be < -2))
```

Более подходящая структура Validated для хранения ошибок
```scala
import cats.Semigroup  
import cats.data.Validated  
import cats.data.Validated._ // for Valid and Invalid  
import cats.syntax.apply._ // for mapN  
import cats.syntax.semigroup._ // for |+| 
  
sealed trait Check[E, A] {  
	import Check._  
	  
	def and(that: Check[E, A]): Check[E, A] = And(this, that)  
	def or(that: Check[E, A]): Check[E, A] = Or(this, that)
	def apply(a: A)(implicit s: Semigroup[E]): Validated[E, A] = this match {  
		case Pure(func) => func(a)  
		case And(left, right) =>  
			(left(a), right(a)).mapN((_, _) => a)  
		case Or(left, right) =>  
			left(a) match {  
				case Valid(a) => Valid(a)  
				case Invalid(e1) =>  
					right(a) match {  
						case Valid(a) => Valid(a)  
						case Invalid(e2) => Invalid(e1 |+| e2)  
					}  
			}
	}  
}  
  
object Check {  
	final case class And[E, A](left: Check[E, A], right: Check[E, A]) extends Check[E, A]  
	final case class Or[E, A](left: Check[E, A], right: Check[E, A]) extends Check[E, A]
	final case class Pure[E, A](func: A => Validated[E, A]) extends Check[E, A]  
	  
	def pure[E, A](f: A => Validated[E, A]): Check[E, A] = Pure(f)  
}

val c: Check[List[String], Int] = Check.pure { v =>  
	if (v > 2) valid(v)  
	else invalid(List("Must be > 2"))  
}  
  
val d: Check[List[String], Int] = Check.pure { v =>  
	if (v < -2) valid(v)  
	else invalid(List("Must be < -2"))  
}  
val check2: Check[List[String], Int] = c and d  
  
check2(5) // Invalid(List(Must be < -2))
check2(0) // Invalid(List(Must be > 2, Must be < -2))
```

Переименуем Check в Predicate, обладающий свойством идентичности:
для р = `Predicate[E, A]` и элементов а1 и a2 типа A, если p(a1) == Success(a2) тогда a1 == a2

```scala
import cats.Semigroup  
import cats.data.Validated  
import cats.data.Validated._ // for Valid and Invalid  
import cats.syntax.apply._ // for mapN  
import cats.syntax.semigroup._ // for |+|  
  
sealed trait Predicate[E, A] {  
	import Predicate._  
	  
	def and(that: Predicate[E, A]): Predicate[E, A] = And(this, that)  
	def or(that: Predicate[E, A]): Predicate[E, A] = Or(this, that)  
	def apply(a: A)(implicit s: Semigroup[E]): Validated[E, A] = this match {  
		case Pure(func) => func(a)  
		case And(left, right) => (left(a), right(a)).mapN((_, _) => a) 
		case Or(left, right) =>  
			left(a) match {  
				case Valid(a) => Valid(a)  
				case Invalid(e1) =>  
					right(a) match {  
						case Valid(a) => Valid(a)  
						case Invalid(e2) => Invalid(e1 |+| e2)  
					}  
			}  
	}  
}  
  
object Predicate {  
	final case class And[E, A](left: Predicate[E, A], right: Predicate[E, A]) extends Predicate[E, A]  
	final case class Or[E, A](left: Predicate[E, A], right: Predicate[E, A]) extends Predicate[E, A]  
	final case class Pure[E, A](func: A => Validated[E, A]) extends Predicate[E, A]  
	  
	def pure[E, A](f: A => Validated[E, A]): Predicate[E, A] = Pure(f)  
}
```

Окончательный вариант
```scala
import cats.Semigroup  
import cats.data.Validated  
import cats.data.Validated._ // for Valid and Invalid  
import cats.syntax.apply._ // for mapN  
import cats.syntax.semigroup._ // for |+|  
import cats.syntax.validated._ // for valid and invalid  
  
sealed trait Predicate[E, A] {  
	import Predicate._  
	  
	def and(that: Predicate[E, A]): Predicate[E, A] = And(this, that)  
	  
	def or(that: Predicate[E, A]): Predicate[E, A] = Or(this, that)  
	  
	def apply(a: A)(implicit s: Semigroup[E]): Validated[E, A] = this match {  
	case Pure(func) => func(a)  
	case And(left, right) =>  
		(left(a), right(a)).mapN((_, _) => a)  
			case Or(left, right) =>  
				left(a) match {  
					case Valid(a) => Valid(a)  
					case Invalid(e1) =>  
						right(a) match {  
							case Valid(a) => Valid(a)  
							case Invalid(e2) => Invalid(e1 |+| e2)  
				}  
		}  
	}  
}  
  
object Predicate {  
	final case class And[E, A](left: Predicate[E, A], right: Predicate[E, A]) extends Predicate[E, A]  
	final case class Or[E, A](left: Predicate[E, A], right: Predicate[E, A]) extends Predicate[E, A]  
	final case class Pure[E, A](func: A => Validated[E, A]) extends Predicate[E, A]  
	  
	def apply[E, A](f: A => Validated[E, A]): Predicate[E, A] = Pure(f)  
	  
	def lift[E, A](err: E, fn: A => Boolean): Predicate[E, A] = Pure(a => if (fn(a)) a.valid else err.invalid)  
}  
  
sealed trait Check[E, A, B] {  
	import Check._  
	def apply(in: A)(implicit s: Semigroup[E]): Validated[E, B]  
	  
	def map[C](f: B => C): Check[E, A, C] = Map[E, A, B, C](this, f)  
	  
	def flatMap[C](f: B => Check[E, A, C]) = FlatMap[E, A, B, C](this, f)  
	  
	def andThen[C](that: Check[E, B, C]): Check[E, A, C] = AndThen[E, A, B, C](this, that)  
}  
  
object Check {  
	final case class FlatMap[E, A, B, C](check: Check[E, A, B], func: B => Check[E, A, C]) extends Check[E, A, C] {  
	def apply(a: A)(implicit s: Semigroup[E]): Validated[E, C] = check(a).withEither(_.flatMap(b => func(b)(a).toEither))  
	}  
	  
	final case class Map[E, A, B, C](check: Check[E, A, B], func: B => C) extends Check[E, A, C] {  
	def apply(a: A)(implicit s: Semigroup[E]): Validated[E, C] = check(a) map func  
	}  
	  
	final case class AndThen[E, A, B, C](check: Check[E, A, B], next: Check[E, B, C]) extends Check[E, A, C] {  
	def apply(a: A)(implicit s: Semigroup[E]): Validated[E, C] = check(a).withEither(_.flatMap(b => next(b).toEither))  
	}  
	  
	final case class Pure[E, A, B](func: A => Validated[E, B]) extends Check[E, A, B] {  
	def apply(a: A)(implicit s: Semigroup[E]): Validated[E, B] = func(a)  
	}  
	  
	final case class PurePredicate[E, A](pred: Predicate[E, A]) extends Check[E, A, A] {  
	def apply(a: A)(implicit s: Semigroup[E]): Validated[E, A] =  
	pred(a)  
	}  
	  
	def apply[E, A](pred: Predicate[E, A]): Check[E, A, A] = PurePredicate(pred)  
	def apply[E, A, B](func: A => Validated[E, B]): Check[E, A, B] = Pure(func)  
}  
  
import cats.data.NonEmptyList  
type Errors = NonEmptyList[String]  
  
def error(s: String): NonEmptyList[String] =  
NonEmptyList(s, Nil)  
  
def longerThan(n: Int): Predicate[Errors, String] =  
Predicate.lift(  
error(s"Must be longer than $n characters"),  
str => str.size > n)  
  
val alphanumeric: Predicate[Errors, String] =  
Predicate.lift(  
error(s"Must be all alphanumeric characters"),  
str => str.forall(_.isLetterOrDigit))  
  
def contains(char: Char): Predicate[Errors, String] = Predicate.lift(  
error(s"Must contain the character $char"),  
str => str.contains(char))  
  
def containsOnce(char: Char): Predicate[Errors, String] = Predicate.lift(  
error(s"Must contain the character $char only once"),  
str => str.filter(c => c == char).size == 1)  
  
// A username must contain at least four characters  
// and consist entirely of alphanumeric characters  
val checkUsername: Check[Errors, String, String] =  
Check(longerThan(3) and alphanumeric)  
  
// An email address must contain a single `@` sign.  
// Split the string at the `@`.  
// The string to the left must not be empty.  
// The string to the right must be  
// at least three characters long and contain a dot.  
val splitEmail: Check[Errors, String, (String, String)] = Check(_.split('@') match {  
	case Array(name, domain) => (name, domain).validNel[String]  
	case _ => "Must contain a single @ character".invalidNel[(String, String)]  
})  
val checkLeft: Check[Errors, String, String] = Check(longerThan(0))  
val checkRight: Check[Errors, String, String] = Check(longerThan(3) and contains('.'))  
val joinEmail: Check[Errors, (String, String), String] = Check {  
	case (l, r) =>  
	(checkLeft(l), checkRight(r)).mapN(_ + "@" + _)  
}  
val checkEmail: Check[Errors, String, String] = splitEmail andThen joinEmail  
  
final case class User(username: String, email: String)  
  
def createUser(username: String, email: String): Validated[Errors, User] =  
(checkUsername(username), checkEmail(email)).mapN(User)  
  
createUser("Noel", "noel@underscore.io") // Valid(User(Noel,noel@underscore.io))  
createUser("", "dave@underscore.io@io") // Invalid(NonEmptyList(Must be longer than 3 characters, Must contain a single @ character))
```

# Kleisli
Применяет к переданному значению функцию `A => F[B]` и позволяет композировать функции
Является алиасом для монады ReaderT.
```scala
import cats.data.Kleisli  
  
val step1: Kleisli[List, Int, Int] = Kleisli(x => List(x + 1, x - 1))  
val step2: Kleisli[List, Int, Int] = Kleisli(x => List(x, -x))  
val step3: Kleisli[List, Int, Int] = Kleisli(x => List(x * 2, x / 2))  
  
step1.run(1) // List(2, 0) 
step2.run(1) // List(1, -1)
step3.run(1) // List(2, 0)
  
val pipeline = step1 andThen step2 andThen step3  
  
pipeline.run(1) // List(4, 1, -4, -1, 0, 0, 0, 0)
```
Перепишем наш пример в терминах
```scala
import cats.Semigroup  
import cats.data.Validated._ // for Valid and Invalid  
import cats.data.{Kleisli, NonEmptyList, Validated}  
import cats.instances.either._ // for Semigroupal  
import cats.syntax.apply._ // for mapN  
import cats.syntax.validated._ // for valid and invalid  
  
type Errors = NonEmptyList[String]  
type Result[A] = Either[Errors, A]  
type Check[A, B] = Kleisli[Result, A, B]  
  
sealed trait Predicate[E, A] {  
	import Predicate._  
	  
	def and(that: Predicate[E, A]): Predicate[E, A] = And(this, that)  
	  
	def apply(a: A)(implicit s: Semigroup[E]): Validated[E, A] = this match {  
		case Pure(func) => func(a)  
		case And(left, right) =>  
			(left(a), right(a)).mapN((_, _) => a)  
	}  
	  
	def run(implicit s: Semigroup[E]): A => Either[E, A] = (a: A) => this (a).toEither  
}  
  
object Predicate {  
	final case class And[E, A](left: Predicate[E, A], right: Predicate[E, A]) extends Predicate[E, A]  
	  
	final case class Pure[E, A](func: A => Validated[E, A]) extends Predicate[E, A]  
	  
	def lift[E, A](err: E, fn: A => Boolean): Predicate[E, A] = Pure(a => if (fn(a)) a.valid else err.invalid)  
}  
  
def error(s: String): NonEmptyList[String] = NonEmptyList(s, Nil)  
  
// Create a check from a function:  
def check[A, B](func: A => Result[B]): Check[A, B] = Kleisli(func)  
  
// Create a check from a Predicate:  
def checkPred[A](pred: Predicate[Errors, A]): Check[A, A] = Kleisli[Result, A, A](pred.run)  
  
def longerThan(n: Int): Predicate[Errors, String] =  
	Predicate.lift(  
		error(s"Must be longer than $n characters"),  
		str => str.size > n)  
  
val alphanumeric: Predicate[Errors, String] =  
	Predicate.lift(  
		error(s"Must be all alphanumeric characters"),  
		str => str.forall(_.isLetterOrDigit))  
  
def contains(char: Char): Predicate[Errors, String] = Predicate.lift(  
	error(s"Must contain the character $char"),  
	str => str.contains(char))  
  
val checkUsername: Check[String, String] =  
	checkPred(longerThan(3) and alphanumeric)  
  
val splitEmail: Check[String, (String, String)] =  
	check(_.split('@') match {  
		case Array(name, domain) =>  
			Right((name, domain))  
		case _ =>  
			Left(error("Must contain a single @ character"))  
	})  
  
val checkLeft: Check[String, String] = checkPred(longerThan(0))  
  
val checkRight: Check[String, String] = checkPred(longerThan(3) and contains('.'))  
val joinEmail: Check[(String, String), String] =  
	check {  
		case (l, r) =>  
			(checkLeft(l), checkRight(r)).mapN(_ + "@" + _)  
	}  
val checkEmail: Check[String, String] = splitEmail andThen joinEmail  
  
final case class User(username: String, email: String)  
  
def createUser(username: String, email: String): Either[Errors, User] = (  
	checkUsername.run(username),  
	checkEmail.run(email)  
).mapN(User)  
  
createUser("Noel", "noel@underscore.io") // Right(User(Noel,noel@underscore.io))  
createUser("", "dave@underscore.io@io") // Left(NonEmptyList(Must be longer than 3 characters))
```
# CRDTs
```scala
final case class GCounter(counters: Map[String, Int]) {  
	def increment(machine: String, amount: Int) = {  
		val value = amount + counters.getOrElse(machine, 0)  
		GCounter(counters + (machine -> value))  
	}  
	  
	def merge(that: GCounter): GCounter =  
		GCounter(that.counters ++ this.counters.map {  
			case (k, v) =>  
				k -> (v max that.counters.getOrElse(k, 0))  
		})  
	  
	def total: Int = counters.values.sum  
}
```

BoundedSemiLattice
```scala
import cats.kernel.CommutativeMonoid  
  
object wrapper {  
	trait BoundedSemiLattice[A] extends CommutativeMonoid[A] {  
		def combine(a1: A, a2: A): A  
		def empty: A  
	}  
	  
	object BoundedSemiLattice {  
		implicit val intInstance: BoundedSemiLattice[Int] =  
			new BoundedSemiLattice[Int] {  
				def combine(a1: Int, a2: Int): Int =  
					a1 max a2  
				val empty: Int = 0  
			}  
		  
		implicit def setInstance[A]: BoundedSemiLattice[Set[A]] = 
			new BoundedSemiLattice[Set[A]] {  
				def combine(a1: Set[A], a2: Set[A]): Set[A] =  
					a1 union a2  
		  
				val empty: Set[A] = Set.empty[A]  
			}  
	}  
}
```

Абстрагируемся
```scala
import cats.syntax.semigroup._ // for |+|  
import cats.syntax.foldable._ // for combineAll  
  
final case class GCounter[A](counters: Map[String, A]) {  
	def increment(machine: String, amount: A)  
	(implicit m: CommutativeMonoid[A]): GCounter[A] = {  
		val value = amount |+| counters.getOrElse(machine, m.empty)  
		GCounter(counters + (machine -> value))  
	}  
	  
	def merge(that: GCounter[A])  
	(implicit b: BoundedSemiLattice[A]): GCounter[A] =  
		GCounter(this.counters |+| that.counters)  
	  
	def total(implicit m: CommutativeMonoid[A]): A =  
		this.counters.values.toList.combineAll  
}
```

Абстракция
```scala
import cats.syntax.semigroup._ // for |+|  
import cats.syntax.foldable._ // for combineAll  
  
trait GCounter[F[_, _], K, V] {  
	def increment(f: F[K, V])(k: K, v: V)  
	(implicit m: CommutativeMonoid[V]): F[K, V]  
	  
	def merge(f1: F[K, V], f2: F[K, V])  
	(implicit b: BoundedSemiLattice[V]): F[K, V]  
	  
	def total(f: F[K, V])  
	(implicit m: CommutativeMonoid[V]): V  
}  
  
object GCounter {  
	def apply[F[_, _], K, V]  
	(implicit counter: GCounter[F, K, V]) = counter  
	  
	implicit def mapGCounterInstance[K, V]: GCounter[Map, K, V] = new GCounter[Map, K, V] {  
		def increment(map: Map[K, V])(key: K, value: V)  
		(implicit m: CommutativeMonoid[V]): Map[K, V] = {  
			val total = map.getOrElse(key, m.empty) |+| value  
			map + (key -> total)  
		}  
	  
		def merge(map1: Map[K, V], map2: Map[K, V])  
		(implicit b: BoundedSemiLattice[V]): Map[K, V] =  
			map1 |+| map2  
		  
		def total(map: Map[K, V])  
		(implicit m: CommutativeMonoid[V]): V =  
			map.values.toList.combineAll  
	}  
}

val g1 = Map("a" -> 7, "b" -> 3)  
val g2 = Map("a" -> 2, "b" -> 5)  
val counter = GCounter[Map, String, Int]  
val merged = counter.merge(g1, g2) // Map(a -> 7, b -> 5)
val total = counter.total(merged) // 12
```

Абстрагирование Key Value хранилища
```scala
trait KeyValueStore[F[_,_]] {  
	def put[K, V](f: F[K, V])(k: K, v: V): F[K, V]  
	def get[K, V](f: F[K, V])(k: K): Option[V]  
	def getOrElse[K, V](f: F[K, V])(k: K, default: V): V =  
	get(f)(k).getOrElse(default)  
	def values[K, V](f: F[K, V]): List[V]  
}  
  
	implicit val mapKeyValueStoreInstance: KeyValueStore[Map] = new KeyValueStore[Map] {  
	def put[K, V](f: Map[K, V])(k: K, v: V): Map[K, V] = f + (k -> v)  
	  
	def get[K, V](f: Map[K, V])(k: K): Option[V] =  
		f.get(k)  
	  
	override def getOrElse[K, V](f: Map[K, V])  
	(k: K, default: V): V =  
		f.getOrElse(k, default)  
	  
	def values[K, V](f: Map[K, V]): List[V] =  
		f.values.toList  
}  
  
implicit class KvsOps[F[_, _], K, V](f: F[K, V]) {  
	def put(key: K, value: V)  
	(implicit kvs: KeyValueStore[F]): F[K, V] =  
		kvs.put(f)(key, value)  
	  
	def get(key: K)(implicit kvs: KeyValueStore[F]): Option[V] = kvs.get(f)(key)  
	  
	def getOrElse(key: K, default: V)  
	(implicit kvs: KeyValueStore[F]): V =  
		kvs.getOrElse(f)(key, default)  
	  
	def values(implicit kvs: KeyValueStore[F]): List[V] = kvs.values(f)  
}  
  
implicit def gcounterInstance[F[_, _], K, V]  
(implicit kvs: KeyValueStore[F], km: CommutativeMonoid[F[K, V]]) =  
	new GCounter[F, K, V] {  
		def increment(f: F[K, V])(key: K, value: V)  
			(implicit m: CommutativeMonoid[V]): F[K, V] = {  
			val total = f.getOrElse(key, m.empty) |+| value  
			f.put(key, total)  
		}  
  
		def merge(f1: F[K, V], f2: F[K, V])  
		(implicit b: BoundedSemiLattice[V]): F[K, V] =  
			f1 |+| f2  
		  
		def total(f: F[K, V])(implicit m: CommutativeMonoid[V]): V = f.values.combineAll  
}
```