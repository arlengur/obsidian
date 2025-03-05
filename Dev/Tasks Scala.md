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

3. Вывести индексы элементов в массиве сумма которых равна искомому
```scala
def twoSum(nums: Array[Int], target: Int): Array[Int] = {  
	val tmpMap = scala.collection.mutable.Map[Int, Int]()  
	for (i <- nums.indices) {  
		val complement = target - nums(i)  
		if (tmpMap.contains(complement)) {  
			return Array(tmpMap(complement), i)  
		}  
		tmpMap(nums(i)) = i  
	}  
	Array()  
}

// twoSum(Array(2, 7, 11, 15), 9) = 01
// twoSum(Array(3, 2, 4), 6) = 12
// twoSum(Array(3, 3, 2, 4), 6) = 01 
```

4. Последовательно идущие единицы

Требуется найти в бинарном векторе самую длинную последовательность единиц и вывести её длину. Желательно получить решение, работающее за линейное время и при этом проходящее по входному массиву только один раз.
Первая строка входного файла содержит одно число n, n ≤ 10000. Каждая из следующих n строк содержит ровно одно число — очередной элемент массива.
Выходной файл должен содержать единственное число — длину самой длинной последовательности единиц во входном массиве.

Пример:
5
1
0
1
0
1
Вывод: 1

```scala
val n     = scala.io.StdIn.readInt()  
var count = 0  
var max   = 0  
for (_ <- 1 to n) {  
  val i = scala.io.StdIn.readInt()  
  if (i == 1) count += 1 else count = 0  
  if (max < count) max = count  
}  
println(max)
```

5. Найти максимальное количество единиц в массиве
```scala
def findMaxConsecutiveOnes(nums: Array[Int]): Int = {  
	var count = 0  
	var res = 0  
	for (i <- nums.indices) {  
		if (nums(i) == 0) count = 0 else {  
			count += 1  
			res = Math.max(count, res)  
		}  
	}  
	res  
}

// findMaxConsecutiveOnes(Array(1, 1, 0, 1, 1, 1)) // 3
// findMaxConsecutiveOnes(Array(1, 0, 1, 1, 0, 1)) // 2
```

6. Дан список Int разделенных пробелом. Подсчитать количество перемен знака в последовательности, где ноль не перемена. 
	Например: 
	- -1 0 1 = 1 перемена знака
	- -2 -5 10 = 1 перемена знака
	- 11 23 -5 0 = 1 перемена знака
	- 43 0 23 -2 -3 4 -1 = 3 перемены знака
```scala
import scala.io.StdIn.readLine  
val input =  readLine( ).split(" ").flatMap(_.toIntOption).toSeq  
  
def sum(in: Seq[Int]): Int = {  
  val nonZero = in.filter(_ != 0)  
  
  if (nonZero.nonEmpty)  
    (nonZero zip nonZero.tail).collect {  
      case (i1,  i2) if i1.sign * i2.sign < 0 => 1  
    }.sum  
  else 0  
}  
  
println(sum(input))
```

7. Посчитать все последовательности одинаковых символов Ответ выдать в виде `Seq[(Char, Int)]` (символ и число последовательных повторений)
```scala
val in = "Sssstriiingsss"  
  
def solution(in: String): Seq[(Char, Int)] = in.foldLeft(List.empty[(Char, Int)]) {  
	(acc, c) => if(acc.nonEmpty && acc.head._1 == c) (c -> (acc.head._2 + 1)) +: acc.tail else (c -> 1) +: acc  
}
```

8. На вход `Seq[Future[String]]` Получить `Future[(Seq[String], Seq[Throwable])]` - результат агрегации выполненых Future и исключений

```scala
import scala.concurrent.duration.Duration  
import scala.concurrent.{Await, ExecutionContext, Future}  
  
implicit val ec: ExecutionContext = ExecutionContext.global  
  
val talk = Seq(  
	Future {  
		Thread.sleep(1000)  
		"red"  
	},  
	Future.failed(new RuntimeException("exception1")),  
	Future.successful("blue"),  
	Future.failed(new RuntimeException("exception2")),  
	Future.successful("green"),  
	Future.failed(new RuntimeException("exception3"))  
)  
  
val func = Future.sequence(talk.map { f => f.map(Right(_)) }.map(_.recover(Left(_)))).map(f => f.partitionMap(identity))  
  
val result = Await.result(func, Duration.Inf)
```

9. Камни и украшения

Даны две строки строчных латинских символов: строка J и строка S. Символы, входящие в строку J, — «драгоценности», входящие в строку S — «камни». Нужно определить, какое количество символов из S одновременно являются «драгоценностями». Проще говоря, нужно проверить, какое количество символов из S входит в J.
Пример: ab, aabbccd результат 4

```scala
val jewels = scala.io.StdIn.readLine()  
val stones = scala.io.StdIn.readLine()  
  
var count = 0  
for (s <- stones)
  if (jewels.contains(s)) count += 1  
 
println("Count " + count)
```

10. Удаление дубликатов

Дан упорядоченный по неубыванию массив целых 32-разрядных чисел. Требуется удалить из него все повторения.
Желательно получить решение, которое не считывает входной файл целиком в память, т.е., использует лишь константный объем памяти в процессе работы.

Первая строка входного файла содержит единственное число n, n ≤ 1000000.
На следующих n строк расположены числа — элементы массива, по одному на строку. Числа отсортированы по неубыванию.

Выходной файл должен содержать следующие в порядке возрастания уникальные элементы входного массива.

Пример 1:
5
2
4
8
8
8
Вывод:
2
4
8

Пример 2:
5
2
2
2
8
8

Вывод:
2
8

```scala
val n     = scala.io.StdIn.readInt()  
val s   = mutable.TreeSet[Int]()  
for (_ <- 1 to n) {  
  s += scala.io.StdIn.readInt()  
}  
println(s)
```

11. Анаграммы

Даны две строки, состоящие из строчных латинских букв. Требуется определить, являются ли эти строки анаграммами, т. е. отличаются ли они только порядком следования символов.
Входной файл содержит две строки строчных латинских символов, каждая не длиннее 100 000 символов. Строки разделяются символом перевода строки.
Выходной файл должен содержать единицу, если строки являются анаграммами, и ноль в противном случае.

Пример 1:
qiu
iuq
Вывод: 1

Пример 2:
zprl
zprc
Вывод: 0

```scala
val s1 = scala.io.StdIn.readLine()  
val s2 = scala.io.StdIn.readLine()  
println(s1.sorted == s2.sorted)
```

12. Восстановить маршрут

 ```scala
 [(Spb, Moscow), (Spb, Habarovsk), (Barnaul, Habarovsk)] => ["Moscow", "Spb", "Habarovsk", "Barnaul"]
```

1. Get list last

```scala
val l = List(1, 1, 2, 3, 5, 8)  

// l.last builtin
  
def getLast[A](l: List[A]): A = l match {  
  case h::Nil => h  
  case _::t => getLast(t)  
  case _ => throw new NoSuchElementException  
}
```

1. Get list last but one element

```scala
val l = List(1, 1, 2, 3, 5, 8)  

// l.takeRight(2).head
  
def getLast[A](l: List[A]): A = l match {  
  case h::_::Nil => h  
  case _::t => getLast(t)  
  case _ => throw new NoSuchElementException  
}
```

1. Get list nth element

```scala
val l = List(1, 1, 2, 3, 5, 8)  

// l(2)
  
def getNth[A](n:Int, l: List[A]): A = (n,l) match {  
  case (0, h::_) => h  
  case (n, _::t) => getNth(n-1, t)  
  case _ => throw new NoSuchElementException  
}
```

1. Палиндром: 
- Лёша на полке клопа нашёл
- уверен и не реву
- Он — верба, но Она — бревно
- Искать такси
- Лидер бредил
- шалаш
- radar
- 404

```scala
def isPalindrome(s: String): Boolean = {  
  s.replaceAll("\\s", "") == s.replaceAll("\\s", "").reverse  
}

@tailrec  
def isPalindrome(s: String): Boolean = {  
  if (s.isEmpty || s.length == 1)  
    true  
  else    if (s.charAt(0).toLower == s.charAt(s.length - 1).toLower)  
      isPalindrome(s.substring(1, s.length - 1))  
    else  
      false}  
isPalindrome("лёша на полке клопа нашёл".replaceAll("\\s", ""))
```

1. Посчитать элементы в списке

```scala
val words = Array("tea", "coffee", "apple", "coffee", "orange", "tea", "coffee")  
words.groupMapReduce(identity)(_ => 1)(_ + _) 
// Map(orange -> 1, tea -> 2, apple -> 1, coffee -> 3)
```
1. Поворот матрицы на 90 и -90

```scala
def transpose(m: List[List[Int]]): List[List[Int]] = {  
  (m.head.indices toList) map { c =>  
    (m.indices toList) map { r =>  
      m(r)(c)  
    }  
  }  
}  
def reverseRows(m: List[List[Int]]): List[List[Int]] = m.map(_.reverse)  
def reverseCols(m: List[List[Int]]): List[List[Int]] = {  
  (m.indices toList).map { r =>  
    (m.head.indices toList).map { c =>  
      m(m.size-1-r)(c)  
    }  
  }  
}  
  
val m = Array.fill(3,3)(Random.nextInt(9))  
val m1 = transpose(m.map(_.toList).toList)  
val m2 = reverseRows(m1) // 90 rotation  
val m3 = reverseCols(m1) // -90 rotation  
println(m.map(_.mkString(" ")).mkString("\n"))  
println()  
println(m2.map(_.mkString(" ")).mkString("\n"))  
println()  
println(m3.map(_.mkString(" ")).mkString("\n"))
```

1. for-comprehension: результат его работы - это тип из первой строки

```scala
val b = Option(1)  
val a = List(1,2,3)  
  
val c = for {  
  x <- a  
  y <- b  
} yield x + y 
// List(2, 3, 4)

val c = for {  
  x <- b  
  y <- a
} yield x + y 
// type mismatch
```

1. Custom class

```scala
case class NonEmptyList[A](head: A, tail: Option[NonEmptyList[A]]) {  
  
  def toList: List[A] = {  
    tail match {  
      case Some(v) => head +: v.toList  
      case None => List(head)  
    }  
  }  
  
  def toNonEmptyList(l: List[A]): NonEmptyList[A] = {  
    l.tail.foldLeft(NonEmptyList(l.head, None)) {  
      (acc, a) => NonEmptyList(a, Option(acc))  
    }  
  }  
  
  def reverse: NonEmptyList[A] = toNonEmptyList(toList)  
}

case class NonEmptyList[A](head: A, tail: Option[NonEmptyList[A]]) {  
  def rev(l: Option[NonEmptyList[A]], acc: NonEmptyList[A]): NonEmptyList[A] = l match {  
    case Some(v) => rev(v.tail, NonEmptyList(v.head, Option(acc)))  
    case None    => acc  
  }  
  def reverse: NonEmptyList[A] = tail match {  
    case Some(v) => rev(v.tail, NonEmptyList(v.head, Some(NonEmptyList(head, None))))  
    case None    => NonEmptyList(head, None)  
  }  
}

val l =  
  NonEmptyList(1, Option(NonEmptyList(2, Option(NonEmptyList(3, Option(NonEmptyList(4, Option(NonEmptyList(5, None)))))))))
```