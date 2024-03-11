## Scala type system
![](Scala_type_system.svg)
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

Проблемы List:
- Занимает в два раза больше массива
- Много элементов - нагрузка на сборщик мусора
- Время выполнения многих операций пропорционально длине O(N)
- Вставка в конец - только с полным копированием (список раскручивается в стек и потом стек собирается в новый список)
- средства диагностики heap dump плохо работают

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
  sealed trait Platform
  case class IOS(appId: String) extends Platform
  case class Android(packageId: String, sha1Cert: String) extends Platform
В данном случае мы имеем дело с гибридным типом, т.к.

работа идет с платформой iOS или Android ("или" указывает на Sum ADT);
Product ADT был задействован при описании того же Android т.к. пришлось создавать универсальный конструктор, учитывающий различные значения для packageId и sha1Cert.


Еще примеры

Фиксированная схема данных и произвольные операции, тогда Pattern Matching:
Product - тип можно разобрать на пары, а потом обратно собрать.
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

Конструкция Product with Serializable необходимо чтобы была возможность добавить такой элемент в Buffer.

Пример:
val number = Number(3)
val expr = Plus(Number(2), Number(3))
val buf = ArrayBuffer(Number(2), expr)
buf += number

Конструкция final case class запрещает наследование помогает компилятору не искать что-то где-то еще.

или полиморфизм, когда фиксированные операции и большое разнообразие объектов:

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
# Задачки

1.  Функцию batch traverse которая будет для `Seq[Int], f: Int => Future[Int]`выдавать `Future[Seq[Int]]` в которой Future над элементами будут запускаться не сразу а батчами размера size
```scala
object BatchTraverse extends App {  
implicit val ec = ExecutionContext.global  
  
def batchTraverse1(in: Seq[Int], size: Int)(f: Int => Future[Int]): Future[Seq[Int]] =  
	in.grouped(size).toSeq.foldLeft(Future.successful(Seq.empty[Int])){  
		case (acc, l) => acc.flatMap(x => Future.sequence(l.map(f)).map(y => x ++ y) )  
}  
  
def batchTraverse2(in: Seq[Int], size: Int)(f: Int => Future[Int]): Future[Seq[Int]] =  
	in.grouped(size).toSeq.foldLeft(Future.successful(Seq.empty[Int])) {  
		case (acc, l) => for {  
			y <- acc  
			x <- Future.sequence(l.map(f))  
		} yield y ++ x  
}  
 
def func(i: Int): Future[Int] = Future {  
	println(s"running func($i)")  
	Thread.sleep(1500)  
	i * 100  
}  
  
val in = 1 to 12  
  
val res = Await.result(batchTraverse1(in, 3)(func), Duration.Inf)  
println(res)  
}
```

2. Бинарное дерево (имплементировать fold и подсчитать сумму в узлах)
```scala
object BinaryTreeFold extends App {  
	sealed trait Tree  
	case class Node(v: Int, left: Tree, right: Tree) extends Tree  
	case object Nil extends Tree  
	  
	object Tree {  
		def fold[B](tree: Tree, zero: B)(f: (Int, B, B) => B): B = tree match {  
			case Node(v, left, right) => f(v, fold(left, zero)(f), fold(right, zero)(f))  
			case Nil => zero  
		}  
	}  
	import Tree._  
	val in = Node(1, Node(3, Nil, Node(1, Nil, Nil)), Node(2, Nil, Nil))  
	val sum = fold(in, 1){  
		(v, l, r) => v * l * r  
	}  
	println(in)  
	println(sum)  
}
```