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

4. Найти максимальное количество единиц в массиве
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

5. Дан список Int разделенных пробелом. Подсчитать количество перемен знака в последовательности, где ноль не перемена. 
	Например: 
	- -1 0 1 = 1 перемена знака
	- -2 -5 10 = 1 перемена знака
	- 11 23 -5 0 = 1 перемена знака
	- 43 0 23 -2 -3 4 -1 = 3 перемены знака
```scala
import scala.io.StdIn.readLine  
  
val input = readLine().split(' ').flatMap(_.toIntOption).toSeq  
  
def sum(in: Seq[Int]): Int = {  
	val nonZero = in.filter(_ != 0)  
	  
	if (nonZero.nonEmpty)  
		(nonZero zip nonZero.tail).collect { case (i1, i2) if i1 * i2 < 0 => 1 }.sum  
	else 0  
}  
  
println(sum(input))
```

6. Посчитать все последовательности одинаковых символов Ответ выдать в виде `Seq[(Char, Int)]` (символ и число последовательных повторений)
```scala
val in = "Sssstriiingsss"  
  
def solution(in: String): Seq[(Char, Int)] = in.foldLeft(List.empty[(Char, Int)]) {  
	(acc, c) => if(acc.nonEmpty && acc.head._1 == c) (c -> (acc.head._2 + 1)) +: acc.tail else (c -> 1) +: acc  
}
```

7. На вход `Seq[Future[String]]` Получить `Future[(Seq[String], Seq[Throwable])]` - результат агрегации выполненых Future и исключений

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